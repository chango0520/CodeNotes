# 蓝桥杯单片机 键盘&数码管 代码实操



## 按键

### 独立按键

只有最左侧一列按键

最常规的写法：

```c
void key_task(){
    if(P32 == 0){
        Delay_ms(50);
        if(P32 == 0){ //延迟一会儿在判断，消抖
            //执行命令
            while(P32 ==0); //防止一直按住
            //执行命令
            Delay_ms(50);
        }
    }
}
```

还可以参考矩阵键盘写法

```c
unsigned char Key_Scan()
{
	unsigned char key_value = 0;
	
	// 独立按键模式下，S4~S7 直接对应 P33~P30 引脚 (J5需短接GND)
	if (P33 == 0) 
	{
		key_value = 4; // S4 按下
	}
	else if (P32 == 0) 
	{
		key_value = 5; // S5 按下
	}
	else if (P31 == 0) 
	{
		key_value = 6; // S6 按下
	}
	else if (P30 == 0) 
	{
		key_value = 7; // S7 按下
	}
	
	return key_value;
}
```



### 矩阵键盘

**Key_Scan() **先进行键盘的扫描，确定按下按键的编号

```c
unsigned char Key_Scan()
{
	unsigned int temp;
	unsigned char value;
	
	P3 |= 0x0f;
	
	P44 = 0; P42 = 1; P35 = 1;
	temp = P3 & 0x0f;
	P44 = 1; P42 = 0; P35 = 1;
	temp = (temp << 4) | (P3 & 0x0f);
	P44 = 1; P42 = 1; P35 = 0;
	temp = (temp << 4) | (P3 & 0x0f);
	P44 = 1; P42 = 1; P35 = 1;
	temp = (temp << 4) | (P3 & 0x0f);
	
	switch(~temp)
	{
		case 0x8000: value = 4; break;
		case 0x4000: value = 5; break;
		case 0x0800: value = 8; break;
		case 0x0400: value = 9; break;
		case 0x0080: value = 12; break;
		case 0x0040: value = 13; break;
		default: value = 0; break;
	}
	return value;
}

```



键盘任务函数 **Key_Proc()** 通过三段式消抖，同时获取时上升沿还是下降沿（最常见的作用就是区分长短按）

```c
void Key_Proc()
{
	static unsigned char old = 0;
	unsigned char up,down,temp;
	if(cnt_key < 50) return;
	cnt_key = 0;
	
	temp = Key_Scan();
	down = temp & (temp ^ old); //这一刻才按下，即下降沿
	up   = ~temp & (temp ^ old); //这一刻松手了，即上升沿
	old = temp; //获取上一刻
	
	if(down)
	{
		switch(down)
		{
			case 9:
				if(flag_ui == 1) flag_9 = 1;
			break;
		}
	}
	
	if(up)
	{
		switch(up)
		{
			case 9:
				if(temp_para > 0) temp_para--;
				if(cnt_2s >= 2000) {......} //长按超过2s
				cnt_2s = 0;
			break;
		}
	}
}
```



最后记得在定时器中断函数中写：

```c
if(cnt_key < 50) cnt_key++; //具体数值按照实际来
```

还要记得把 **Key_Proc()** 加入主函数运行



## 数码管



先写**Nixie_Show()** 实现数码管动态显示

```c
//内存不够，可以放到pdata
pdata unsigned char Nixie_Value[] = {0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff};
code unsigned char Seg_Table[] = {0xC0, 0xF9, 0xA4, 0xB0, 0x99, 0x92, 0x82, 0xF8,
									0x80, 0x90, 0x88, 0x83, 0xC6, 0xA1, 0x86, 0x8E};
void Nixie_Show()
{
	static unsigned char com; 
	Y7C(0xff); //先全部熄灭
	Y6C(0x01 << com); //逐步选中
	Y7C(Nixie_Value[com]); //显示特定内容
	if(++com == 8) com = 0; //从头再来
}
```



同理再写数码管任务函数

```c
void Nixie_Proc()
{
	if(cnt_nixie < 100) return; //未计满不执行函数
	cnt_nixie = 0; //执行即清零
	
	memset(Nixie_Value, 0xff, 8); //清除之前的显示内容，防止干扰
	if(flag_ui ==0){
		Nixie_Value[0] = 0x86;
		Nixie_Value[5] = 0xbf;
		Nixie_Value[3] = Seg_Table[Temperature / 10];
		Nixie_Value[4] = Seg_Table[Temperature % 10];
	}
}
```



下面就和键盘一样的