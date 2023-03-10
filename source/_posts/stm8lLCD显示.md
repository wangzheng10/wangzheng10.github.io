---
title: stm8l LCD显示
date: 2021/12/29
categoreies:
- 单片机
- stm8s
tags:
- 单片机
- stm8s
---

如图所示，可以看到LCD配置寄存器包含4个控制寄存器LCD_CRx、频率选择寄存器LCD_FRQ、端口掩码寄存器LCD_PM、LCD显示寄存器LCD_RAM。

![LCD寄存器](https://img-blog.csdnimg.cn/a6fd4b2902174ba9838c8eec139e3c09.png#pic_center)

---
## LCD初始化配置
---

stm8L的LCD显示
初始化配置分为以下几个步骤：
1.	背光LED灯初始化
2.	使能LCD外设时钟
3.	LCD参数设置
4.	LCD端口设置
5.	LCD对比度设置

---

注：使用寄存器库将指针->更改为下划线_即可。u8_t即unsigned char类型。
```c
void HW_Init_LcdIO(void) 
{  
  /* configure PB5, Output and push-pull and Low level  */
  GPIOB->DDR |= 0x20;
  GPIOB->CR2 &= ~0x20;
  GPIOB->ODR &= ~0x20;
  GPIOB->CR1 |= 0x20;
  
  /*Enables the LCD peripheral clock*/
  CLK->PCKENR2 |= 0x08;
  
  /*lcd configure */   
  LCD->FRQ = 0x30;      /* LCD_Prescaler_8,LCD_Divider_16 */
  LCD->CR1 &= ~0x06;    /* Clear the duty bits */
  LCD->CR4 &= ~0x02;   /* Clear the DUTY8 bit */
  LCD->CR1 |= 0x02;    /* Configure the Duty cycle 1/2 */
  LCD->CR1 |= 0x00;    /* Configure the Bias cycle 1/3 */
  LCD->CR2 &= ~0x01;    /* Clear the voltage source bit */
  LCD->CR2 |= 0x01;   /* External voltage source for the LCD */
  
  /*LCD port configuration*/ 
  LCD->PM[0x00] = 0xff;           /* seg 0-7 */
  LCD->PM[0x01] = 0x3f;          /* seg 8,9 12,13*/
  LCD->PM[0x02] = 0x00; 
  LCD->PM[0x03] = 0x00;
  
  /*Configure LCD Contrast*/
  LCD->CR2 |= 0x0E;
  /*Configure LCD DeadTime*/
  LCD->CR3 |= 0x01;
  /*Configures the LCD pulses on duration*/
  LCD->CR2 |= 0xA0;
  /* Enable Lcd */
  LCD->CR3 |= 0x40;
}

```

---

## LCD段码配置

---

首先查看LCD规格书中的段码的对应表，本文采用的LCD如下图所示：
![LCD段码](https://img-blog.csdnimg.cn/3a080aa1d7f444b3864f946f29d4e79c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iul5oKy5rWq,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

实际LCD使用的端口是COM0和COM1，查看stm8l参考手册的LCD寄存器表：
![LCD寄存器图表](https://img-blog.csdnimg.cn/77d4a8c35d6641f28b1361a6e842c7e3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iul5oKy5rWq,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

如果想要点亮图标L、M、H，从上图可以看到L对应COM1的seg1[0]，M对应seg1[1]，H对应COM0的seg0[1]。

因此将LCD_RAM3的高位第0位置1，就可以点亮L，图标显示代码如下：

```c
void HW_Set_LcdShowIcon(IN u8_t nLcdIconNo,IN u8_t nValue)
{
	switch (nLcdIconNo)
	{
		case eLcdIcon_L:
			   if (nValue)
				 {
					 LCD->RAM[LCD_RAMRegister_3] |= 0x10;
				 }
				 else
				 {
					 LCD->RAM[LCD_RAMRegister_3] &= (~0x10);
				 }
				 break;
		case eLcdIcon_M:
				 if (nValue)
				 {
					 LCD->RAM[LCD_RAMRegister_3] |= 0x20;
				 }
				 else
				 {
					 LCD->RAM[LCD_RAMRegister_3] &= (~0x20);
				 }
				 break;
		case eLcdIcon_H:
		     if (nValue)
				 {
					 LCD->RAM[LCD_RAMRegister_0] |= 0x02;
				 }
				 else
				 {
					 LCD->RAM[LCD_RAMRegister_0] &= (~0x02);
				 }
				 break;
		case eLcdIcon_COL:
		     if (nValue)
				 {
					 LCD->RAM[LCD_RAMRegister_4] |= 0x04;
				 }
				 else
				 {
					 LCD->RAM[LCD_RAMRegister_4] &= (~0x04);
				 }
				 break;
		case eLcdIcon_PM:
		     if (nValue)
				 {
					 LCD->RAM[LCD_RAMRegister_5] |= 0x02;
				 }
				 else
				 {
					 LCD->RAM[LCD_RAMRegister_5] &= (~0x02);
				 }
				 break;
		case eLcdIcon_S1:
		     if (nValue)
				 {
					 LCD->RAM[LCD_RAMRegister_1] |= 0x20;
				 }
				 else
				 {
					 LCD->RAM[LCD_RAMRegister_1] &= (~0x20);
				 }
				 break;
		default:
		     break;
	}
}

```

注：LCD_RAMRegister_x 为宏定义，对应的是数字x。

---



### 七段数码管数字和字母显示

 ![数码管例图](https://img-blog.csdnimg.cn/8bb76b5ae87442719ec43fd5e4a2f241.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iul5oKy5rWq,size_11,color_FFFFFF,t_70,g_se,x_16#pic_center)

我们将数码管各段命名，并对其进行宏定义：
``` c
#define  a       0x01     
#define  b       0x02     
#define  c       0x04     
#define  d       0x08    
#define  e       0x10     
#define  f       0x20    
#define  g       0x40  
#define NONE     0x00 
```

然后根据所需的数字和字母进行段码组合：
``` c
const u8_t LCD_Num[]=    
{ 
  {a+b+c+d+e+f},               //0   
  {c+b},                       //1   
  {a+b+d+e+g},                 //2   
  {a+c+b+d+g},                 //3   
  {c+b+f+g},                   //4   
  {a+c+f+d+g},                 //5   
  {a+c+e+f+d+g},               //6   
  {a+c+b},                     //7   
  {a+c+e+b+f+d+g},             //8   
  {a+c+b+f+d+g},               //9 
  {b+c+d+e+f},                 //U
  {a+d+e+f+g},                 //E
  {a+b+c+e+f+g},               //R
  {d+e+f},                     //L
  {b+c+e+f+g},                 //H
  {0x00},                      //NULL
};
```
此时，对于每一个种数字和字母，组合的数都是唯一的，不会出现重复的情况。

---

现在我们将第二个数码管的7个段对应的寄存器位置表示出来：

``` 
2C: seg0[5]  RAM0[5]
2D: seg0[2]  RAM0[2]
2E: seg0[3]  RAM0[3]
2G: seg0[4]  RAM0[4]
2A: seg1[4]  RAM4[0]
2B: seg1[5]  RAM4[1]
2F: seg1[3]  RAM3[7]
```

---

这样我们输入一个数num，比如1，LCD_Num[1] = {c+b}，此时c和b对应的第1和第2位将置1，然后我们将这个值赋给RAM对应的位置即可点亮2C和2D，将会显示数字1。显然实际2B2C和我们定义的数组中的bc位置不是一一对应的，所以我们需要根据实际情况进行移位操作。

比如c在我们的宏定义中是第2位，而2C是在RAM0的第5位，所以将其左移3位即可。

``` c
void HW_LCD_WriteNum2(u8_t nNum) 
{ 
  /* COM0-3: first page 0x00 */
  /* COM4-7: second page 0x04 */
  LCD->CR4 = 0x00; //The LCD RAM is selected as the first page
  //COM0  
  LCD->RAM[LCD_RAMRegister_0] &= (~0x3c);  
  LCD->RAM[LCD_RAMRegister_0] |= (u8_t)((LCD_Num[nNum]<< 3) & 0x20);// 2C
  LCD->RAM[LCD_RAMRegister_0] |= (u8_t)((LCD_Num[nNum]>> 1) & 0x04);// 2D
  LCD->RAM[LCD_RAMRegister_0] |= (u8_t)((LCD_Num[nNum]>> 1) & 0x08);// 2E
  LCD->RAM[LCD_RAMRegister_0] |= (u8_t)((LCD_Num[nNum]>> 2) & 0x10);// 2G
  //COM1
  LCD->RAM[LCD_RAMRegister_3] &= (~0x80);
  LCD->RAM[LCD_RAMRegister_3] |= (u8_t)((LCD_Num[nNum]<< 2) & 0x80);// 2F

  LCD->RAM[LCD_RAMRegister_4] &= (~0x03);
  LCD->RAM[LCD_RAMRegister_4] |= (u8_t)(LCD_Num[nNum] & 0x03);// 2AB
}
```

我们将7个段码依次进行赋值，即可显示对应的数字和字母，这样简单的显示程序就写好了。可以根据自己需求进行进一步包装优化。
