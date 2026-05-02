# 蓝桥杯单片机 超声波 代码实操

超声波作用就是**测距**

先把引脚补全

```c
sbit Tx = P1^0;
sbit Rx = P1^1;
```

然后就可以写测距函数了

```c
unsigned int csb_collect(){
	unsigned int distance;
	unsigned char num = 10;
	
	TR0 = 0;
	TH0 = 0xff;
	TL0 = 0xf4; //设置初值，实现发送 40kHz的超声波
	TR0 = 1;
	
	while(num--)
	{
		while(!TF0);
		TF0 = 0;
		Tx ^= 1; //取反发射引脚 （产生方波）
	}
	
	TR0 = 0; //停止计数
	TH0 = 0; //计数清零
	TL0 = 0;
	TR0 = 1;
	
	while(Rx && !TF0);
	TR0 = 0;
	
	if(TF0) //如果计数器跑满还没收到回波
	{
		distance = 999; //设置一个错误数值
		TF0 = 0;
	}
	else
	{
		distance = ((TH0<<8)|TL0) * 0.017; //dist = t*340/2 ， 340m/s == 0.034cm/us
	}
	
	return distance;
	
}
```

使用的话

```c
dist = csb_collect();
```

