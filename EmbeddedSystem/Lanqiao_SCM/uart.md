# 蓝桥杯单片机 串口 代码实操

类似于定时器，先初始化（一般我们采用定时器2）

```c
void Uart1_Init(void)	//9600bps@12.000MHz
{
	SCON = 0x50;		//8位数据,可变波特率
	AUXR |= 0x01;		//串口1选择定时器2为波特率发生器
	AUXR &= 0xFB;		//定时器时钟12T模式
	T2L = 0xE6;			//设置定时初始值
	T2H = 0xFF;			//设置定时初始值
	AUXR |= 0x10;		//定时器2开始计时
	ES = 1;				//使能串口1中断
}

```

引用库

```c
#include <stdio.h>
```



putchar函数（用于通过uart发送数据，给printf调用）

```c
extern char putchar(char ch){
    SBUF=ch; //相当于把数据给SBUF发送就会调用写SBUF
    while(!TI); ////一个字符8位，软件等待硬件执行完毕
    TI=0;
    return ch;
}
```



需要几个参数来协助实现功能

```c
unsigned char uart_buf[10]; //缓冲区
unsigned char uart_index; //正在读取的位
unsigned char cnt_uart; //定时器中断计数
big flag_uart; //开启计数的标志位
```

在定时器中断内写

```c
if(flag_uart) cnt_uart++;
```



然后写串口中断函数

```c
void Uart1_Isr(void) interrupt 4
{	//发送中断似乎一般用不到，都是上位机发送命令控制单片机
	//if (TI)				//检测串口1发送中断
	//{
		//TI = 0;			//清除串口1发送中断请求位
	//}
	if (RI)				//检测串口1接收中断
	{	
        flag_uart=1; //开始计时
        uart_buf[uart_index] = SBUF; //把SBUF里面的数据读取出来
        uart_index++;
		RI = 0;			//清除串口1接收中断请求位
        //防止缓冲区溢出
        if(uart_index>10){
            memset(uart_buf,0,10);
            uart_index=0;
        }
	}
}
```

再写串口任务函数

```c
void uart_task(){
    if(!uart_index) return;
    if(cnt_uart>=10){
        cnt_uart=flag_uart=0;
        //此处写具体要实现的功能
        if(uart_buf[0] = '0' && uart_buf[1] = 'k') printf("This is Right!");
        //例如还能用sscanf接收数据，2026省赛就是考了这个
        memset(uart_buf,0,uart_index);
        uart_index=0;
    }
    
}
```

如果上位机发送"temp 35",采用sscanf来做

sscanf的用法是

```c
sscanf(源字符串，格式字符串， 变量地址列表);
```



```c
if(sscanf((char*)uart_buf, "temp %d", &temp)==1) target_temp=temp;
```

非常优雅