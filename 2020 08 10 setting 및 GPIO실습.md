# 2020 08 10
=============

1. 프로젝트 만든 후 설정

프로젝트 마우스 우클릭 -> properties -> Tools Paths

build tools folder
C:\STM32_eclipse\STM32Toolchain\Build Tools\2.6-201507152002\bin

Toolchian folder
C:\STM32_eclipse\STM32Toolchain\gnu-arm\5.4 2016q3\bin


2. 
STM32CubeMX Untitled프로그램 실행

STM32F405RGTx 칩 선택

pinout탭에서 RCC - High speed Clock -> Crystal로 변경 ( 사용하겠다는 뜻 )

3. 

STM32CubeMX Clock Configuration설정을 바꾼 후 추출한 main.c파일 부분
void SystemClock_Config(void)함수를

이클립스 _initialize_hardware.c의 SystemClock_Config함수와 바꿔치기한다


4. GPIO실습 : LED불 켜기
```
// ---------관계 레지스터의 주소-------------------------------------------------------
#define RCC_AHB1ENR (*(volatile unsigned long*)0x40023830)	//RCC_AHB1 GPIO clock enable
#define GPIOC_MODER (*(volatile unsigned long*)0x40020800)	//GPIOC port mode 레지스터
#define GPIOC_OTYPER (*(volatile unsigned long*)0x40020804)	//GPIOC output type 레지스터
#define GPIOC_OSPEEDR (*(volatile unsigned long*)0x40020808)	//GPIO output speed 레지스터
#define GPIOC_PUPDR (*(volatile unsigned long*)0x4002080C)	//GPIOC pull-up/pull-down 레지스터
#define GPIOC_ODR (*(volatile unsigned long*)0x40020814)	//GPIOC 출력레지스터

void my_ms_delay_int_count(volatile unsigned int nTime)
{
	nTime = (nTime * 14000);
	for( ; nTime > 0; nTime--);
	//오실로스코프를 이용한 실측값이 14000이 1ms가 된다.
}

int main(int argc, char* argv[])
{
	//-----------GPIO초기화 (각 레지스터의 값 )-----------------
	RCC_AHB1ENR = 0x00000004;	//GPIOC에 RCC clock 공급
	GPIOC_MODER = 0x00000010;	//GPIOC(2)를 general purpose output mode로
	GPIOC_OTYPER = 0;			//GPIOC(2)는 output push-pull
	GPIOC_PUPDR = 0;
	GPIOC_OSPEEDR = 0;			//GPIOC(2)를 low speed로

	//----------메인 루틴에서 하는 일-------------------------
	while (1)
	{
		//Active low 이기때문에 0이 on 1이 off

		//LED1 (PC2) off
		GPIOC_ODR = 0x0004; //0b0000 0000 0000 0100

		//대기 sleep (sleep이 없기 때문에 공회전을 시킨다 )
		my_ms_delay_int_count(1000);

		//LED1 (PC2) on
		GPIOC_ODR = 0x0000; //0b0000 0000 0000 0000

		//대기 sleep
		my_ms_delay_int_count(1000);
	}
}
```


* 위와 같은 내용이지만 헤더파일을 이용해 코딩을 바꾸기
```
#include "stm32F4xx.h"

void my_ms_delay_int_count(volatile unsigned int nTime)
{
	nTime = (nTime * 14000);
	for( ; nTime > 0; nTime--);
	//오실로스코프를 이용한 실측값이 14000이 1ms가 된다.
}

int main(int argc, char* argv[])
{
	//-----------GPIO초기화 (각 레지스터의 값 )-----------------
	RCC->AHB1ENR = 0x00000004;	//GPIOC에 RCC clock 공급
	GPIOC->MODER = 0x00000010;	//GPIOC(2)를 general purpose output mode로
	GPIOC->OTYPER = 0;			//GPIOC(2)는 output push-pull
	GPIOC->PUPDR = 0;
	GPIOC->OSPEEDR = 0;			//GPIOC(2)를 low speed로

	//----------메인 루틴에서 하는 일-------------------------
	while (1)
	{
		//Active low 이기때문에 0이 on 1이 off

		//LED1 (PC2) off
		GPIOC->ODR = 0x0004; //0b0000 0000 0000 0100

		//대기 sleep (sleep이 없기 때문에 공회전을 시킨다 )
		my_ms_delay_int_count(1000);

		//LED1 (PC2) on
		GPIOC->ODR = 0x0000; //0b0000 0000 0000 0000

		//대기 sleep
		my_ms_delay_int_count(1000);
	}
}
```

5. GPIO실습 : DCMotor

```
#include "stm32F4xx.h"

void my_ms_delay_int_count(volatile unsigned int nTime)
{
	nTime = (nTime * 14000);
	for( ; nTime > 0; nTime--);
	//오실로스코프를 이용한 실측값이 14000이 1ms가 된다.
}

int main(int argc, char* argv[])
{
    //GPIO초기화 (각 레지스터의 값 )
    RCC->AHB1ENR = 0x00000003;      //GPIOA, GPIOB에 RCC clock공급
    GPIOA->MODER |= 0x00001000;     //GPIOA6을 general purpose output mode로
    GPIOA->OTYPER = 0;              //GPIOA6은 output push-pull
    GPIOA->PUPDR = 0;
    GPIOA->OSPEEDR = 0;             //GPIOA6을 low speed로

    GPIOB->MODER = 0x00005000;      //GPIOB6, GPIOB7을 general purpose output mode로
    GPIOB->OTYPER = 0;              //GPIOB6, GPIOB7은 output push-pull
    GPIOB->PUPDR = 0;               
    GPIOB->OSPEEDR = 0;             //GPIOB6, GPIOB7을 low speed

    //DCMotor_PWM은 항상 High
    GPIOA->ODR = 0x0040;

    //-----------메인루틴에서 하는일-------------
    while(1)
    {
        GPIOB->ODR = 0x0040;        //CW
        my_ms_delay_int_count(1000); //1s
        GPIOB->ODR = 0x0000;        //Stop
        my_ms_delay_int_count(1000); //1s
    }
}
```
6. GPIO실습 : Piezo

```
#include "stm32F4xx.h"

int main(int argc, char* argv[])
{
    //GPIO초기화 ( 각 레지스터의 값 )
    RCC->AHB1ENR = 0x00000002;      //GPIOB에 RCC clock공급
    GPIOB->MODER = 0x40000000;      //GPIOB(15)를 general purpose output mode로
    GPIOB->OTYPER = 0;              //GPIOB(15)는 output push-pull
    GPIOB->PUPDR = 0;
    GPIOB->OSPEEDR=0;

    //-----------메인루틴에서 하는 일 ---------------
    unsigned int period, buf;
    while(1)
    {
        for(period=0x1000; period>=1; period--)
        {
            GPIOB->ODR = 0x8000;        //Piezo on
            buf = period;
            while(buf--);

            GPIOB->ODR = 0x0000;        //Piezo off
            buf = period;
            while(buf--);
        }
    }   
}
```