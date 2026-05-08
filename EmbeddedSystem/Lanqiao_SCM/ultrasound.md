# 蓝桥杯单片机 超声波 代码实操

超声波作用就是**测距**

先要初始化定时器1

```c
void Timer1_Init(void)		//1毫秒@12.000MHz
{
	AUXR &= 0xBF;			//定时器时钟12T模式
	TMOD &= 0x0F;			//设置定时器模式
	TR1 = 1;				//定时器1开始计时
}

```



先把引脚补全

```c
sbit Tx = P1^0;
sbit Rx = P1^1;
```

然后就可以写测距函数了

```c
unsigned int csb_collect(){
	unsigned int distance;
	unsigned char num=10;
	
	TR1=0;
	TH1=0xff;
	TL1=0xf4;
	TR1=1;
	
	while(num--){
		while(!TF1);
		TF1=0;
		Tx^=1;
	}
	
	TR1=0;
	TH1=TL1=0;
	TR1=1;
	
	while(Rx && !TF1);
	TR1=0;
	
	if(TF1){
		distance =999;
		TF1=0;
	}
	else{
		distance = ((TH1<<8)|TL1) * 0.017;
	}
	
	return distance;
}
```

使用的话

```c
dist = csb_collect();
```

