---
title: STM32实现PWM
date: 2018-05-31 19:06:00
tags: STM32
---

---

简介：PWM，全称 Pulse Width Modulation，即脉冲宽度调制本次，实验是通过配置stm32f103c8中的PB5引脚来输出PWM的

配置流程如下：
> 1、初始化GPIO
> 2、初始化定时器
> 3、设置TIM3_CH2的PWM模式，使能TIM3的CH输出
> 4、使能定时器

遇到问题及解决：

- 程序烧写进开发板无现象，原因：暂未找到原因，怀疑是boot启动问题。换了一块开发板，能得到正常现象
- 没有得到PWM波形，原因：配置不正确，没有正确初始化GPIO，我这里是少了个重映射，也没有使能定时器
- 呼吸灯闪烁速率太快，原因：分频设置有问题，改为0
- 配置多个PWM时，如果配置部分映射模式，会出现其中一个引脚无法输出PWM波形，原因：原因暂不清楚，改成完全映射模式即可
<br><br>
单个PWM代码：
```c
#include "stm32f10x.h"

#define LED_ON GPIO_SetBits(GPIOA, GPIO_Pin_1);
#define LED_OFF GPIO_ResetBits(GPIOA, GPIO_Pin_1);

void delay(uint32_t t)
{
	uint16_t i;
	while(t--)
		for(i = 0; i < 1000; i++);
}

//GPIO初始化
void gpio_init()
{
	GPIO_InitTypeDef GPIO_InitStructure1, GPIO_InitStructure2;
	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB | RCC_APB2Periph_AFIO, ENABLE);		//使能GPIOB引脚，使能AFIO时钟（定时器3通道2需要重映射到PB5引脚）
	
	GPIO_InitStructure1.GPIO_Pin = GPIO_Pin_5;	//PB5引脚
	GPIO_InitStructure1.GPIO_Speed = GPIO_Speed_50MHz;	//速率配置为50MHz
	GPIO_InitStructure1.GPIO_Mode = GPIO_Mode_AF_PP;	//复用推挽功能
	GPIO_Init(GPIOB, &GPIO_InitStructure1);		//初始化GPIOB端口
	
	GPIO_PinRemapConfig(GPIO_PartialRemap_TIM3, ENABLE);	//重映射
	
	/*GPIO_InitStructure2.GPIO_Pin = GPIO_Pin_1;
	GPIO_InitStructure2.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_InitStructure2.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_Init(GPIOA, &GPIO_InitStructure2);
	GPIO_ResetBits(GPIOA, GPIO_Pin_1);*/
	
}

//定时器初始化
void timer_init()
{
	TIM_TimeBaseInitTypeDef tim;	//定义定时器初始化结构体
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);	//使能定时器3
	
	tim.TIM_Period = 19999;	//自动重装载寄存器的值
	tim.TIM_Prescaler = 0;		//TIMX预分频的值，设为0表示不分频
	tim.TIM_ClockDivision = 0;	//时钟分割
	tim.TIM_CounterMode = TIM_CounterMode_Up;	//向上计数
	TIM_TimeBaseInit(TIM3, &tim);	//对定时器进行初始化
}

//设置TIM_CH2的PWM模式，使能TIM的CH2输出
void pwm_init()
{
	TIM_OCInitTypeDef  TIM_OCInitStructure;	//定义结构体
	TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM2;	//选择定时器模式，PWM模式2
	TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;	//比较输出使能
	TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;		//输出比较极性高
	TIM_OC2Init(TIM3, &TIM_OCInitStructure);						//初始化定时器模式
	TIM_OC2PreloadConfig(TIM3, TIM_OCPreload_Enable);			//使能定时器在CCR2上的预装载值
}

int main()
{
	int i;
	gpio_init();
	timer_init();
	pwm_init();
	TIM_Cmd(TIM3,ENABLE); 	//使能定时器TIM3
	while(1)
	{		
		for(i = 19999; i > 0; i--)
		{
			delay(1);
			TIM_SetCompare2(TIM3, i);
		}
		
		for(i = 0; i < 19999; i++)
		{
			delay(1);
			TIM_SetCompare2(TIM3, i);		
		}
		
	}
}
```
<br><br>
多个PWM代码：
```c
#include "stm32f10x.h"
#include "func.h"

void delay(uint32_t t)
{
	uint16_t i;
	while(t--)
		for(i = 0; i < 1000; i++);
}

int KEY_Scan(int SW)
{
	static unsigned char key_up = 1;
	
	if(key_up && (GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_0) == 0))
	{
		delay(200);
		key_up = 0;
		
		if(GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_0) == 0)
		{
			if(SW != 1)
				SW = 1;
			else
				SW = 2;
		}
		
		/*if(GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_0) == 0)
		{
			if(SW != 1 && SW != 0)
				SW = 1;
			else
				SW = 2;
		}
		else
			SW = 0;*/
	}
	else if(key_up && (GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_1) == 0))
	{
		delay(200);
		key_up = 0;
		
		if(GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_1) == 0)
		{
			if(SW != 3)
				SW = 3;
			else
				SW = 4;
		}
	}		
	else
	{
		SW = 0;
		key_up = 1;	
	}
	return SW;
}

void gpio_init(void)
{
	GPIO_InitTypeDef GPIO_InitStructure1;
	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC | RCC_APB2Periph_AFIO, ENABLE);
	
	GPIO_InitStructure1.GPIO_Pin = GPIO_Pin_6 | GPIO_Pin_7;
	GPIO_InitStructure1.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_InitStructure1.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_Init(GPIOC, &GPIO_InitStructure1);
	
	GPIO_PinRemapConfig(GPIO_FullRemap_TIM3, ENABLE);
	
	/*GPIO_InitStructure2.GPIO_Pin = GPIO_Pin_1;
	GPIO_InitStructure2.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_InitStructure2.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_Init(GPIOA, &GPIO_InitStructure2);
	GPIO_ResetBits(GPIOA, GPIO_Pin_1);*/
	
}

void key_init(void)
{
	GPIO_InitTypeDef GPIO_KEY;
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	
	
	GPIO_KEY.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_KEY.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1;
	GPIO_KEY.GPIO_Speed = GPIO_Speed_50MHz;
	
	GPIO_Init(GPIOA, &GPIO_KEY);
}

void timer_init(void)
{
	TIM_TimeBaseInitTypeDef tim;
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
	
	tim.TIM_Period = 999;
	tim.TIM_Prescaler = 719;
	tim.TIM_ClockDivision = 0;
	tim.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInit(TIM3, &tim);
}

void pwm_init(void)
{
	TIM_OCInitTypeDef  TIM_OCInitStructure;
	TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM2;
	TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;
	TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;
	
	TIM_OC1Init(TIM3, &TIM_OCInitStructure);
	TIM_OC1PreloadConfig(TIM3, TIM_OCPreload_Enable);
	
	TIM_OC2Init(TIM3, &TIM_OCInitStructure);
	TIM_OC2PreloadConfig(TIM3, TIM_OCPreload_Enable);
}
```
<br><br>
用STM32的按键控制机械臂，代码如下：
```c
代码：
#include "stm32f10x.h"
#define LED_ON GPIO_SetBits(GPIOA, GPIO_Pin_1);
#define LED_OFF GPIO_ResetBits(GPIOA, GPIO_Pin_1);


void delay(uint32_t t)
{
	uint16_t i;
	while(t--)
		for(i = 0; i < 1000; i++);
}

int KEY_Scan(int SW)
{
	static unsigned char key_up = 1;
	
	if(key_up && (GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_0) == 0))
	{
		delay(200);
		key_up = 0;
		if(GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_0) == 0)
		{
			if(SW != 1 && SW != 0)
				SW = 1;
			else
				SW = 2;
		}
		else
			SW = 0;
	}
	else
	{
		SW = 0;
		key_up = 1;	
	}
	return SW;
}

void gpio_init()
{
	GPIO_InitTypeDef GPIO_InitStructure1, GPIO_InitStructure2;
	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB | RCC_APB2Periph_AFIO, ENABLE);
	
	GPIO_InitStructure1.GPIO_Pin = GPIO_Pin_5;
	GPIO_InitStructure1.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_InitStructure1.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_Init(GPIOB, &GPIO_InitStructure1);
	
	GPIO_PinRemapConfig(GPIO_PartialRemap_TIM3, ENABLE);
	
	/*GPIO_InitStructure2.GPIO_Pin = GPIO_Pin_1;
	GPIO_InitStructure2.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_InitStructure2.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_Init(GPIOA, &GPIO_InitStructure2);
	GPIO_ResetBits(GPIOA, GPIO_Pin_1);*/
	
}

void key_init()
{
	GPIO_InitTypeDef GPIO_KEY;
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	
	
	GPIO_KEY.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_KEY.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1;
	GPIO_KEY.GPIO_Speed = GPIO_Speed_50MHz;
	
	GPIO_Init(GPIOA, &GPIO_KEY);
}

void timer_init()
{
	TIM_TimeBaseInitTypeDef tim;
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
	
	tim.TIM_Period = 999;
	tim.TIM_Prescaler = 719;
	tim.TIM_ClockDivision = 0;
	tim.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInit(TIM3, &tim);
}

void pwm_init()
{
	TIM_OCInitTypeDef  TIM_OCInitStructure;
	TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM2;
	TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;
	TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;
	TIM_OC2Init(TIM3, &TIM_OCInitStructure);
	TIM_OC2PreloadConfig(TIM3, TIM_OCPreload_Enable);
}

int main()
{
	int i;
	int key, SW;
	gpio_init();
	key_init();
	timer_init();
	pwm_init();
	TIM_Cmd(TIM3,ENABLE); 
	while(1)
	{
		key = KEY_Scan(SW);
		
		if(key == 1)
		{
			for(i = 740; i < 930; i++)
			{
				delay(10);
				TIM_SetCompare2(TIM3, i);
			}
			key = 0;
			SW = 1;
		}	
		else if(key == 2)
		{
			for(i = 930; i > 740; i--)
			{
				delay(10);
				TIM_SetCompare2(TIM3, i);
			}
			key = 0;
			SW = 2;
		}				
	}
}
```



---



