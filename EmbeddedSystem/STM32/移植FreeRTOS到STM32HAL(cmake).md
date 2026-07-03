# 移植 FreeRTOS

先从官网或者github仓库下载源码（只下FreeRTOS-kernel即可）





## 一.裁剪

保留

### 1.include

全部保留，其中是FreeRTOS的头文件



 ### 2.portable

cmake就用GCC，mdk用RVDS

再在**GCC/RVDS**下根据mcu选择，例如我是stm32f407vet6，内核就是**ARM-M4F**，只保留这个即可，其他全部删除

**MemMang**下一般使用**heap4**，其余全部删除



### 3.其他

 七个源文件（**croutine，event_groups，list，queue，stream_buffer，tasks，timers**），其余全部删除



## 二.配置

### 1.添加FreeRTOS内核到cmake编译脚本

根目录 CmakeLists.txt添加

```cmake
set(FreeRTOS_Src
    ${CMAKE_SOURCE_DIR}/FreeRTOS/croutine.c
    ${CMAKE_SOURCE_DIR}/FreeRTOS/event_groups.c
    ${CMAKE_SOURCE_DIR}/FreeRTOS/list.c
    ${CMAKE_SOURCE_DIR}/FreeRTOS/queue.c
    ${CMAKE_SOURCE_DIR}/FreeRTOS/stream_buffer.c
    ${CMAKE_SOURCE_DIR}/FreeRTOS/tasks.c
    ${CMAKE_SOURCE_DIR}/FreeRTOS/timers.c
    ${CMAKE_SOURCE_DIR}/FreeRTOS/portable/MemMang/heap_4.c
    ${CMAKE_SOURCE_DIR}/FreeRTOS/portable/GCC/ARM_CM4F/port.c
)

# Add sources to executable
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user sources here
    ${FreeRTOS_Src}
)

# Add include paths
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined include paths
    FreeRTOS/include
    FreeRTOSportable/GCC/ARM_CM4F
)

```



### 2.详细配置

从**example\template_configuration**文件夹内复制**FreeRTOSConfig.h**模板到到include内

设置**configCPU_CLOCK_HZ**为频率（例如168000000对应168MHz)

关闭**configUSE_TICKLESS_IDLE**空闲休眠 （0）

改大任务名字长度**configMAX_TASK_NAME_LEN** （32U)

修改**configTICK_TYPE_WIDTH_IN_BITS** （TICK_TYPE_WIDTH_32_BITS）

修改堆尺寸**configTOTAL_HEAP_SIZE** （20*1024U即20kb)

关闭自定义分配**configAPPLICATION_ALLOCATED_HEAP** （0）

修改任务中断优先级

```c
#define configKERNEL_INTERRUPT_PRIORITY          (15<<4)
#define configMAX_SYSCALL_INTERRUPT_PRIORITY     (5<<4)
#define configMAX_API_CALL_INTERRUPT_PRIORITY    configMAX_SYSCALL_INTERRUPT_PRIORITY
```

打开

```c
#define configUSE_TRACE_FACILITY                1
#define configUSE_STATS_FORMATTING_FUNCTIONS    1
```



修改stm32f4xx_it.c

```c
#include "FreeRTOS.h"

#include "task.h"
```

```c
 extern void vPortSVCHandler(void);

 vPortSVCHandler();
```

```c 
extern void xPortPendSVHandler(void);

 xPortPendSVHandler();
```

```c
extern void xPortSysTickHandler(void);

if(xTaskGetSchedulerState() != taskSCHEDULER_NOT_STARTED)

{

xPortSysTickHandler();

}
```

以上全实现在port.c



然后就可以了



