---
title: stm8s硬件层配置
date: 2021/12/14
categories:

- 单片机
- stm8s
tags:
- 单片机
- stm8s

---

## 时钟配置
---

系统时钟默认高速内部时钟HIS，如果想要切换主时钟，可添加以下配置：
``` c
bool_t  bSucc = eFALSE;
CLK_SWCR |= 0x02;  /* 时钟切换使能 */
CLK_SWR = 0xB4;  /* 选择目标时钟源 0xB4: HSE，0xE1: HIS, 0xD2: LSI */
while(CLK_SWCR & 0x01);  /* 等待时钟切换完成 */
if (CLK_SWCR & 0x08) /* 判断目标时钟源是否准备就绪 */
{
  CLK_SWCR = !0x08;
  bSucc = eTRUE;
}
return bSucc;
```
系统时钟分频，默认是8分频，时钟分频寄存器如图所示：
![时钟分配器](https://img-blog.csdnimg.cn/b7a46df5259446e59cf38696262fbeec.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iul5oKy5rWq,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

---

HIS时钟比其他时钟多一个分配器，可查看时钟树，如图所示：
![时钟树](https://img-blog.csdnimg.cn/e78b81c7208e444c930b1da87e4644e4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iul5oKy5rWq,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

一般默认8分频即可，但如果使用串口通信，则选择1分频，频率太低会丢包。

``` c
CLK_CKDIVR = 0x00;
```
---

## 看门狗配置
---

看门狗用于解决软件卡死跑飞的问题，通俗的说就是计时复位，如果一段时间程序没执行喂狗函数，就会将程序复位，重新开始运行。
我们这里使用独立看门狗，它由一个内部128kHz的LSI阻容振荡器作为时钟源驱动，因此即使是主时钟失效时它仍然照常工作。

---
初始化配置如下：
``` c
void HW_Init_IWDG(void)
{
  /* Enable IWDG */
  IWDG_KR = 0xcc;
  /* relieve PR and PLR write protection */
  IWDG_KR = 0x55;
  /* watchdog counter reload value */
  IWDG_RLR = 0xff;
  /* Set prescaler register to 256, 1.02s */
  IWDG_PR = 0x06;
  /* Reloads IWDG counter with value defined in the reload register */
  IWDG_KR = 0xaa;
}
```

---

喂狗函数如下：
``` c
void IWDG_Feed(void)
{
	IWDG_KR = 0xaa;
}
```
IWDG_Feed();放在while(1)的循环内即可定时喂狗。

---

## 定时器配置
---
定时器分为基本定时器、通用定时器和高级定时器。
![定时器功能比较](https://img-blog.csdnimg.cn/2f07eecc333743e5bbeaf91230cd0a04.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iul5oKy5rWq,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

一般使用通信定时器即可，建议定义一个系统时间标志的定时器，不要用延迟函数，延迟函数是堵塞式的，程序会一直在里面运行，这里使用TIM2。
``` c
void HW_Init_SysTmr(void)
{
  /*Peripheral Clock Enable 1, TIM2 */
  CLK_PCKENR1 |= 0x20;
  /* Time base prescaler = 16 */
  TIM2_PSCR = 0x04;
  /* TIM2's period is 1000 */
  TIM2_ARRH = 0x03;
  TIM2_ARRL = 0xE7;
  /* Enable counter and Auto-Reload Preload */
  TIM2_CR1 |= 0x81;
  /* Counter Register value is 1000*/
  TIM2_CNTRH = 0x03;
  TIM2_CNTRL = 0xE7;
  /*  Updata allowed  */
  TIM2_IER |= 0x01;
}
```

---

定时器初始化后，会1ms产生一次中断，将时间标志位放在中断函数中即可。
``` c
@far @interrupt @svlreg void TIM2_IRQHandler(void)
{
  TIM2_SR1 &= ~0x01; //Clear the flag
  /* Interrupt options follow */
}
```

---

这里使用的是STVD编译器，会自动生成一个中断向量文件stm8_interrupt_vetor.c。
查看mcu数据手册，找到TIM2的中断向量，这里TIM2的中断向量是13，在中断向量文件中的向量表_vectab的/* irq13 */处，更改为`{0x82, TIM2_IRQHandler}`。并在表的上方声明引用外部中断函数：`extern @far @interrupt void TIM2_IRQHandler(void);`
![中断向量表](https://img-blog.csdnimg.cn/5e03a7b0848f4d53823983d2df19d6de.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iul5oKy5rWq,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

---

## IO口配置
---

这个比较简单，总共就5个寄存器，方向寄存器Px_DDR，控制寄存器Px_CR1和Px_CR2，输出寄存器Px_ODR，输入寄存器Px_IDR，x是端口A、B、C…
比如设置PA0口输出：
``` c
PA_DDR |= 0x01;   / * output mode */
PA_CR1 |= 0x01;   / * push-pull */
PA_CR2 |= 0x01;  /* high speed */
```
一般输出使用推挽输出，开漏输出的有额外功耗，且速度较慢，但如果有2个IO口短接的，建议使用开漏输出，防止短路。
输出模式时，对Px_IDR赋值即可输出高低电平。

---

在配置输入模式时，Px_CR2用于配置是否开启外部中断， 置1即可开启输入中断。
常用于睡眠唤醒和PWM信号计数。在开启IO口输入中断后，需要配置ITC中断控制器：

``` c
EXTI_CR1 = 0X03; /* PA: rising and falling edge */
```

![ITC中断控制器](https://img-blog.csdnimg.cn/2389deb484c44629b78faa03c146f21d.png#pic_center)

---

PA端口的中断源是EXTI0，中断向量是3，可以通过软件优先级寄存器ITC_SPRx设置中断优先级，默认是3级，可通过设置ITC_SPR1的第6、7位选择PA口的中断优先级，本文默认3级。

![中断优先级寄存器](https://img-blog.csdnimg.cn/c51aa250908242bdbb701388e07d5ce7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Iul5oKy5rWq,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

---

## ADC模拟数字转换器配置
---

### ADC 初始化

stm8s单片机是10位的分辨率，AD输入信号电压不超过单片机供电电压。配置ADC就不要配置IO口输入模式了，配置代码先贴上：
``` c
void HW_Init_AdcIO(void)
{
	/* Enable the ADC Clock */
	CLK_PCKENR2 |= (1 << 3);
	/* ADC Clock frequency is divided by 1 and continuous conversion mode*/
	ADC_CR1 = 0x02;
	/* Data right aligned and enable scan mode */
	ADC_CR2 |= 0x0A;
	/* Enable the selected ADC1 channel 0. */
	ADC_CSR &= 0xF0;
    /* Disable AIN0 Schmitt tigger */
    ADC_TDRL &= ~0x01;
	/* wake up ADC1 from power down mode */
	ADC_CR1 |= 0x01;
}
```

---

在使用模拟通道的时候，施密特触发器必须被关闭，施密特触发器用于信号的整形和变换，相当于一个门电路，只有高低电平，关闭时有利于降低功耗。
此处转换结果选择的是右对齐，低8位被写入ADC_DRL寄存器，高8位写入ADC_DRH寄存器，读取时必须先读低位，再读高位。

![数据右对齐](https://img-blog.csdnimg.cn/38f453e37c594837a52858190a59e45e.png#pic_center)

---

如果选择左对齐方式，读取时必须先读高位，再读低位。

![数据左对齐](https://img-blog.csdnimg.cn/cf17d24574be4b8dbae131a41acf575a.png#pic_center)

---

### ADC读取
---

初始化函数中首次置位ADON位时，ADC从低功耗模式唤醒。为了启动转换必须第二次使用写指令来置位寄存器的ADON位。ADON位是ADC_CR1的第0位，长时间不用ADC时，可清零进入低功耗模式。
读取代码如下：
``` c
u8_t templ = 0;
u8_t temph = 0;
u16_t wAdDat = 0;
/* Start the ADC software conversion */
ADC_CR1 |= 0x01;
/* End of Conversion */
while((ADC_CSR & 0x80) == 0x00);
ADC_CSR &= ~0x80;
/* Read LSB firstly */
templ = ADC_DRL;
/* Then read MSB */
temph = ADC_DRH;
wAdDat = (u16_t)(templ | (u16_t)(temph << 8));
```
---

## 串口UART配置
---

接收数据输入(UART_RX)和发送数据输出(UART_TX)。
配置的步骤如下：
1.	将TX口设置成输出模式，RX口设置成输入模式；
2.	使能串口UART外设时钟
3.	设置数据长度(8位)
4.	设置停止位(1位)
5.	波特率设置(9600)
6.	使能发送器和接收器
7.	使能接受中断

第一步省略，代码如下：
``` c
CLK_PCKENR1 |= 0x08; /* Enable Uart2 CLK */
UART2_CR1 = 0x00;  /* 8 bit data length */
UART2_CR3 &= ~0x38;/* 1 stop bit, disable sck*/
/* Baud rate: 9600 = 16Mhz/1666 */
UART2_BRR2 = 0x02;
UART2_BRR1 = 0x68;
/* Enable transmitter and receiver */
UART2_CR2 = 0x0C;
/* Enable receive interrupt */
UART2_CR2 |= 0x20;
```

---

stm8s是共用发送接受寄存器UART_DR，发送时将值赋值给该寄存器，接受时从该寄存器读取。该寄存器是由2个寄存器组成，发送寄存器TDR和RDR寄存器。

**TDR寄存器**提供了内部总线和输出移位寄存器之间的并行接口。
**RDR寄存器**提供了输入移位寄存器和内部总线之间的并行接口。

发送代码：
``` c
bool_t  HW_Txd_Data(IN u8_t nUartNO, IN u8_t nData)
{
  bool_t  bSucc = eFALSE;
  switch (nUartNO)
  {   				
    case DBG_UART_NO:
         /* wait for The data in the TDR register is 
         transferred by hardware to the shift register */
         while(!(UART2_SR & 0x80));
         UART2_DR = nData;
         bSucc = eTRUE;
         break;
    
    default:
         break;
  }
  return bSucc;
}
```

---

发送时要等待TDR寄存器的数据已经被转移到移位寄存器，再对UART_DR进行写操作。

这种方法比较简单，适用于数据量较小的情况，当数据量比较大时，可能会出现运行速度变慢的问题。可以使用队列的方法，把发送数据写到队列中，然后将队列中的数据依次发送，用if代替while。

接受中断函数：
``` c
@far @interrupt @svlreg void Uart2_RX_IRQHandler(void)
{
  u8_t nRxData;
  DISABLE_ISR();
  // Check UART rxd interrupt flag
  if (0x20 == (UART2_SR & 0x20)) 
  {									  
    UART2_SR &= ~0x20;
    nRxData = UART2_DR;

    if (s_pfnTRCb[DBG_UART_NO]->m_pfnURxd)
    {
      (*(s_pfnTRCb[DBG_UART_NO]->m_pfnURxd))(nRxData);
    }
  } 
  ENABLE_ISR();
}
```
此处使用回调函数将接收到的数据传到软件层处理。

