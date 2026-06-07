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
#include <string.h> //使用memset
```



putchar函数（用于串口重定向）

```c
char putchar(char ch){
    SBUF=ch; //相当于把数据给SBUF发送就会调用写SBUF
    while(!TI); ////一个字符8位，软件等待硬件执行完毕
    TI=0;
    return ch;
}

// 字符串发送函数
void Uart_SendString(unsigned char * uart_buf)
{
	while(*uart_buf != '\0')
	{
		SBUF = *uart_buf++;
		while(!TI);
		TI = 0;
	}
}	
```



需要几个参数来协助实现功能

```c
unsigned char uart_buf[10]; //缓冲区
unsigned char uart_index; //正在读取的位
unsigned char cnt_uart; //定时器中断计数
bit flag_uart; //开启计数的标志位
```

在定时器中断内写

```c
if(flag_uart) cnt_uart++;
```



然后写串口中断函数

```c
void Uart1_Isr(void) interrupt 4
{	
	if (RI)				//检测串口1接收中断
	{	
        flag_uart=1; //开始计时
        uart_buf[uart_index++] = SBUF; //把SBUF里面的数据读取出来
        cnt_uart=0;
		RI = 0;			//清除串口1接收中断请求位
        //防止缓冲区溢出，其实可以不要
        //if(uart_index>10){
        //    memset(uart_buf,0,10);
        //    uart_index=0;
        //}
	}
}
```

再写串口任务函数

```c
void uart_task(){
    if(!uart_index) return;
    if(cnt_uart>=10) //串口超时解析
    {
        //flag_dev_addr = Prase_Dev_Addr(uart_rec_arr, &uart_rec_dev); 自己写函数也行
        //unsigned int uart_rec_dev 串口解析地址值
        if(sscanf(uart_buf, "#%u?", &uart_rec_dev) == 1 && uart_rec_dev == Dev_Address)               
        {
            // 读取当前时间
            Read_RTC(Time);     
            // 串口输出，两种方法均可
            printf("%u.%ukPa@%bu%bu:%bu%bu", pressure/10%10, pressure%10, Time[0]/10%10, Time[0]%10, Time[1]/10%10, Time[1]%10);
            //sprintf(uart_rec_arr, "%u.%ukPa@%bu%bu:%bu%bu", pressure/10%10, pressure%10, Time[0]/10%10, Time[0]%10, Time[1]/10%10, Time[1]%10);
            Uart_SendString(uart_buf);
            // 闪烁过程中再次收到本机压力查询命令，则重置计时
            //flag_led1_blink = 1;
            //led1_rate = 0;
            //led1_count = 0;

        }
        cnt_uart=flag_uart=0;
        memset(uart_buf,0,uart_index);
        uart_index=0;
    }
    
}
```

如果是自己写函数

```c
/**
  * @brief  解析设备地址：从指定格式的字符串中提取合法的设备地址
  * @note   输入字符串必须严格遵循 #XXX? 格式，XXX为001~100的十进制数字，字符串总长度固定为5
  * @param  arr_rec  输入参数，指向待解析的串口接收数据字符串指针
  * @param  Dev_Addr 输出参数，指向存储解析后设备地址的无符号整型变量指针
  * @retval 1 解析成功，设备地址已赋值完成
  * @retval 0 解析失败（格式错误/字符非法/地址超出范围）
  */
bit Prase_Dev_Addr(unsigned char * arr_rec, unsigned int *Dev_Addr)
{
    unsigned int temp = 0;
    *Dev_Addr = 0;
    if(strlen(arr_rec) != 5) return 0;
    if(arr_rec[0] != '#' || arr_rec[4] != '?') return 0;
    if(arr_rec[1] < '0' || arr_rec[1] > '9' ||
       arr_rec[2] < '0' || arr_rec[2] > '9' ||
       arr_rec[3] < '0' || arr_rec[3] > '9')    return 0;
    temp = (arr_rec[1] - '0') * 100 + (arr_rec[2] - '0') * 10 + (arr_rec[3] - '0');
    if(temp < 1 || temp > 100) return 0;
    *Dev_Addr = temp;
    return 1;
}
```

