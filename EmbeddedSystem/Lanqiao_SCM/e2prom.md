# 蓝桥杯单片机 AT24C02 代码实操



e2prom可以实现掉电不掉数据

和PCF8591类似，都是基于I2C协议

```c
//写入eeprom
void W_e2prom(unsigned char dat, unsigned char address)
{
	I2CStart();
	I2CSendByte(0xA0); //解除写保护
	I2CWaitAck();
	I2CSendByte(address);
	I2CWaitAck();
	I2CSendByte(dat);
	I2CWaitAck();
	I2CStop();
    //共计六个延时和Delay_ms效果相同，但是Delay_ms需要重新定义函数很麻烦
	I2C_Delay(255);
	I2C_Delay(255);
	I2C_Delay(255);
	I2C_Delay(255);
	I2C_Delay(255);
	I2C_Delay(255);
    // Delay_ms(5);
}
//读出eeprom
unsigned char R_e2prom(unsigned char address)
{
	unsigned char temp;
	I2CStart();
	I2CSendByte(0xa0);
	I2CWaitAck();
	I2CSendByte(address);
	I2CWaitAck();
	//后面其实和PCF8591差不多
	I2CStart();
	I2CSendByte(0xa1);
	I2CWaitAck();
	temp = I2CReceiveByte();
	I2CSendAck(1);
	I2CStop();
	return temp;
}
```

使用例如

```c
    temp = R_eeprom(0x00);  // ①
    if(temp != 0xAA)        // ②
    {
        // 延时一定不要忘记了， 当前在函数内部延时了
        W_eeprom(0xAA, 0x00);
        W_eeprom(0, 0x01);
        W_eeprom(90, 0x02);
        W_eeprom(8, 0x03);
    }
    else                    // ③
    {
        PL = R_eeprom(0x01);
        PH = R_eeprom(0x02);
        Dev_Address = R_eeprom(0x03);
        PH_Display = PH;
        PL_Display = PL;
    }
```

