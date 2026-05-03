# 蓝桥杯单片机 AT24C02 代码实操



e2prom可以实现掉电不掉数据

和PCF8591类似，都是基于I2C协议

```c
/*发送数据函数*/
void WriteData(unsigned char address,unsigned char dat)
{
		I2C_Start();
		I2C_SendByte(0xA0); //发送器件地址(写模式)
		I2C_ReceiveAck(); //等待应答
		I2C_SendByte(address); //发送eeporm存储地址
		I2C_ReceiveAck();
    
		I2C_SendByte(dat); //发送数据
		I2C_ReceiveAck();
		I2C_Stop();
}


 
/*读取数据函数*/
unsigned char ReadData(unsigned char address)
{
		unsigned char mid = 0;
		I2C_Start();
		I2C_SendByte(0xA0);
		I2C_ReceiveAck();
		I2C_SendByte(address);
		I2C_ReceiveAck();
	
		I2C_Start();
		I2C_SendByte(0xA1); //发送器件地址(读模式)
		I2C_ReceiveAck();
	
		mid = I2C_RecieveByte();
		I2C_SendAck(1);
		I2C_Stop();
	
		return mid;
}
```

使用例如

```c
WriteData(0x00,0x05);//向EEPROM地址0写入数据5
Delay(5);
WriteData(0x01,0x06);//向EEPROM地址1写入数据6
```

