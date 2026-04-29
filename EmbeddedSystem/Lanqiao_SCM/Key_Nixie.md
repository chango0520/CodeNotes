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



### 矩阵键盘

**Key_Scan() **先进行键盘的扫描，确定按下按键的编号，以下代码只使用了两列（因为大部分只用两列，不少**外设使用会屏蔽按键**，例如NE555使用时，SIGNAL和P34连接，会屏蔽第四列）

```c
unsigned char Key_Scan()
{
	unsigned char key_temp;
	unsigned char key_value;
	
	P3 |= 0x0f; //先把行全部拉高
	P44 = 0; P42 = 1; //拉低第一列
	key_temp = P3; //获取第一列按下的
	P44 = 1; P42 = 0;  //同理拉低第二列
	key_temp = (key_temp << 4) | (P3 & 0x0f); //合并两次获取的数据为8位（两列四行）
	
	switch(~key_temp) //取反看得直观一些
	{
		case 0x80: key_value = 4; break; //根据图，S4是1列4行
		case 0x40: key_value = 5; break;
		case 0x08: key_value = 8; break;
		case 0x04: key_value = 9; break; 
		default:key_value = 0; break; //若一开始未赋值，此处切记赋值
	}
	return key_value; //返回按下的按键进行消抖/长短按处理
}
```



键盘任务函数 **Key_Proc()** 通过三段式消抖，同时获取时上升沿还是下降沿（最常见的作用就是区分长短按）

```c
void Key_Proc()
{
	static unsigned char key_old = 0;
	unsigned char key_up,key_down,key_temp;
	if(cnt_key < 50) return;
	cnt_key = 0;
	
	key_temp = Key_Scan();
	key_down = key_temp & (key_temp ^ key_old); //这一刻才按下，即下降沿
	key_up   = ~key_temp & (key_temp ^ key_old); //这一刻松手了，即上升沿
	key_old = key_temp; //获取上一刻
	
	if(key_down)
	{
		switch(key_down)
		{
			case 9:
				if(flag_ui == 1) flag_9 = 1;
			break;
		}
	}
	
	if(key_up)
	{
		switch(key_up)
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
//其中是1-9
code unsigned char Seg_Table[] = {0xc0, 0xf9, 0xa4, 0xb0, 0x99, 0x92, 0x82, 0xf8, 
								0x80, 0x90, 0x88, 0x83, 0xc6, 0xa1, 0x86, 0x8e };
//显示缓冲
unsigned char Nixie_Value[] = {0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff};
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