# 2020 08 12
=============

1. 여러개 외부 인터럽트 실습

* stm32f4xx_it.h
```
#include "stm32f4xx_hal.h"

void EXTI0_IRQHandler(void);
void EXTI1_IRQHandler(void);
void EXTI2_IRQHandler(void);
```

* stm32f4xx_it.c
```
#include "stm32f4xx_hal.h"

void EXTI0_IRQHandler(void)
{
	HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_0);
}
void EXTI1_IRQHandler(void)
{
	HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_1);
}
void EXTI2_IRQHandler(void)
{
	HAL_GPIO_EXTI_IRQHandler(GPIO_PIN_2);
}
```

* main.c
```
#include "stm32f4xx_hal.h"		//관련 레지스터의 주소 지정
#include "stm32f4xx_it.h"		//인터럽트 사용에 필요한 헤더파일

GPIO_InitTypeDef LED;
GPIO_InitTypeDef JOG_CEN;

void EXTILine0_Config(void)
{
	//Enable GPIOB clock
	__HAL_RCC_GPIOB_CLK_ENABLE();

	//configure PB2 pin as input floating
	JOG_CEN.Pin = GPIO_PIN_0;
	JOG_CEN.Mode = GPIO_MODE_IT_RISING;
	JOG_CEN.Pull = GPIO_NOPULL;
	JOG_CEN.Speed = GPIO_SPEED_HIGH;
	HAL_GPIO_Init(GPIOB, &JOG_CEN);

	//enable and set EXTI Line2 Interrupt to the lowest priority
	HAL_NVIC_SetPriority(EXTI0_IRQn, 2, 0);
	HAL_NVIC_EnableIRQ(EXTI0_IRQn);
}
void EXTILine1_Config(void)
{
	//Enable GPIOB clock
	__HAL_RCC_GPIOB_CLK_ENABLE();

	//configure PB2 pin as input floating
	JOG_CEN.Pin = GPIO_PIN_1;
	JOG_CEN.Mode = GPIO_MODE_IT_RISING;
	JOG_CEN.Pull = GPIO_NOPULL;
	JOG_CEN.Speed = GPIO_SPEED_HIGH;
	HAL_GPIO_Init(GPIOB, &JOG_CEN);

	//enable and set EXTI Line2 Interrupt to the lowest priority
	HAL_NVIC_SetPriority(EXTI1_IRQn, 2, 0);
	HAL_NVIC_EnableIRQ(EXTI1_IRQn);
}
void EXTILine2_Config(void)
{
	//Enable GPIOB clock
	__HAL_RCC_GPIOB_CLK_ENABLE();

	//configure PB2 pin as input floating
	JOG_CEN.Pin = GPIO_PIN_2;
	JOG_CEN.Mode = GPIO_MODE_IT_RISING;
	JOG_CEN.Pull = GPIO_NOPULL;
	JOG_CEN.Speed = GPIO_SPEED_HIGH;
	HAL_GPIO_Init(GPIOB, &JOG_CEN);

	//enable and set EXTI Line2 Interrupt to the lowest priority
	HAL_NVIC_SetPriority(EXTI2_IRQn, 2, 0);
	HAL_NVIC_EnableIRQ(EXTI2_IRQn);
}


//인터럽트 루틴
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
	if(GPIO_Pin == GPIO_PIN_0)		//key up
	{
		HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_2);
		HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_3);
	}
	else if(GPIO_Pin == GPIO_PIN_1)		//key down
	{
		HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_2);
	}
	else if(GPIO_Pin == GPIO_PIN_2)		//key cent
	{
		HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_3);
	}

}

int main(int argc, char* argv[])
{
	//enable GPIOA clock
	__HAL_RCC_GPIOA_CLK_ENABLE();		//LED(PA2) LED(PA3)

	//configure PA2, PA3 IO in output push-pull mode
	LED.Pin = GPIO_PIN_2 | GPIO_PIN_3;
	LED.Mode = GPIO_MODE_OUTPUT_PP;
	LED.Pull = GPIO_NOPULL;
	LED.Speed = GPIO_SPEED_HIGH;
	HAL_GPIO_Init(GPIOA, &LED);

	//configure EXTI Line2
	EXTILine0_Config();
	EXTILine1_Config();
	EXTILine2_Config();

	while (1);
}
```

2. 타이머를 이용한 실습

* stm32f4xx_it.h
```
// include--------------------
#include "stm32f4xx_hal.h"

//타이머 인터럽트 처리 핸들러 함수 정의
void TIM2_IRQHandler(void);
```

* stm32f4xx_it.c
```
#include "stm32f4xx_hal.h"
#include "stm32f4xx_it.h"

//TIM인터럽트 ISR을 위한 TimHandle 변수를 외부정의 변수로 선언
extern TIM_HandleTypeDef TimHandle;

//타이머 인터럽트 처리 핸들러 함수 구현
void TIM2_IRQHandler(void)
{
	HAL_TIM_IRQHandler(&TimHandle);
}
```

* main.c
```
#include "stm32f4xx_hal.h"		//관련 레지스터의 주소 지정
#include "stm32f4xx_it.h"		//인터럽트 사용에 필요한 헤더파일

TIM_HandleTypeDef TimHandle;		//타이머의 초기화용 구조체 변수를 선언
GPIO_InitTypeDef LD1;				//GPIO의 초기화를 위한 구조체 변수를 선언

//TIMER의 초기설정용 함수 선언
void TIMER_Config(void)
{
	//Timx Peripheral clock enable
	__HAL_RCC_TIM2_CLK_ENABLE();

	//Set Timx instance
	TimHandle.Instance = TIM2;					//TIM2 사용
	TimHandle.Init.Period = 10000-1;			//업데이트 이벤트 발생시 ARR=9999로 설정
	TimHandle.Init.Prescaler = 8400-1;			//Prescaler = 8399로 설정
	TimHandle.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;		//division을 사용하지 않음
	TimHandle.Init.CounterMode = TIM_COUNTERMODE_DOWN;			//Down Counter 모드 설정
	HAL_TIM_Base_Init(&TimHandle);

	//start the tim base generation in interrupt mode
	HAL_TIM_Base_Start_IT(&TimHandle);		//start channel1

	//configure the NVIC for TIMx
	HAL_NVIC_SetPriority(TIM2_IRQn, 0, 0);		//set interrupt group priority
	HAL_NVIC_EnableIRQ(TIM2_IRQn);				//Enable the TIMx global interrupt
}

//TIM 인터럽트 Callback 함수 : Period Elapsed callback
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef* htim)
{
	HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_2);
	HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_3);
	HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_4);
}

int main(int argc, char* argv[])
{
	//enable GPIOA clock
	__HAL_RCC_GPIOA_CLK_ENABLE();

	//configure PC2 IO in output push-pull mode to driv external LED
	LD1.Pin = GPIO_PIN_2 | GPIO_PIN_3 | GPIO_PIN_4;
	LD1.Mode = GPIO_MODE_OUTPUT_PP;
	LD1.Pull = GPIO_NOPULL;
	LD1.Speed = GPIO_SPEED_LOW;
	HAL_GPIO_Init(GPIOA, &LD1);

	//configure TIMER
	TIMER_Config();

	//메인루틴에서 하는일
  	while (1);
}
```

* 주의사항 빌드시 문제발생하기 때문에
* 타이머 관련 HAl파일 추가

<img src ="/img/20200812_01.PNG" width="300px" height="500px"></img>

<img src ="/img/20200812_02.PNG" width="500px" height="400px"></img>


3. 타이머 2개 사용하기 실습

* stm32f4xx_it.h
```
// include--------------------
#include "stm32f4xx_hal.h"

//타이머 인터럽트 처리 핸들러 함수 정의
void TIM2_IRQHandler(void);
void TIM3_IRQHandler(void);

```

* stm32f4xx_it.c
```
#include "stm32f4xx_hal.h"
#include "stm32f4xx_it.h"

//TIM인터럽트 ISR을 위한 TimHandle 변수를 외부정의 변수로 선언
extern TIM_HandleTypeDef TimHandle2, TimHandle3;

//타이머 인터럽트 처리 핸들러 함수 구현
void TIM2_IRQHandler(void)
{
	HAL_TIM_IRQHandler(&TimHandle2);
}
void TIM3_IRQHandler(void)
{
	HAL_TIM_IRQHandler(&TimHandle3);
}
```

* main.c
```
#include "stm32f4xx_hal.h"		//관련 레지스터의 주소 지정
#include "stm32f4xx_it.h"		//인터럽트 사용에 필요한 헤더파일

TIM_HandleTypeDef TimHandle2, TimHandle3;		//타이머의 초기화용 구조체 변수를 선언
GPIO_InitTypeDef LD1;				//GPIO의 초기화를 위한 구조체 변수를 선언

//TIMER의 초기설정용 함수 선언
void TIMER2_Config(void)
{
	//Timx Peripheral clock enable
	__HAL_RCC_TIM2_CLK_ENABLE();

	//Set Timx instance
	TimHandle2.Instance = TIM2;					//TIM2 사용
	TimHandle2.Init.Period = 10000-1;			//업데이트 이벤트 발생시 ARR=9999로 설정
	TimHandle2.Init.Prescaler = 8400-1;			//Prescaler = 8399로 설정
	TimHandle2.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;		//division을 사용하지 않음
	TimHandle2.Init.CounterMode = TIM_COUNTERMODE_DOWN;			//Down Counter 모드 설정
	HAL_TIM_Base_Init(&TimHandle2);

	//start the tim base generation in interrupt mode
	HAL_TIM_Base_Start_IT(&TimHandle2);		//start channel1

	//configure the NVIC for TIMx
	HAL_NVIC_SetPriority(TIM2_IRQn, 0, 0);		//set interrupt group priority
	HAL_NVIC_EnableIRQ(TIM2_IRQn);				//Enable the TIMx global interrupt
}
void TIMER3_Config(void)
{
	//Timx Peripheral clock enable
	__HAL_RCC_TIM3_CLK_ENABLE();

	//Set Timx instance
	TimHandle3.Instance = TIM3;					//TIM2 사용
	TimHandle3.Init.Period = 5000-1;			//업데이트 이벤트 발생시 ARR=9999로 설정
	TimHandle3.Init.Prescaler = 8400-1;			//Prescaler = 8399로 설정
	TimHandle3.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;		//division을 사용하지 않음
	TimHandle3.Init.CounterMode = TIM_COUNTERMODE_DOWN;			//Down Counter 모드 설정
	HAL_TIM_Base_Init(&TimHandle3);

	//start the tim base generation in interrupt mode
	HAL_TIM_Base_Start_IT(&TimHandle3);		//start channel1

	//configure the NVIC for TIMx
	HAL_NVIC_SetPriority(TIM3_IRQn, 0, 0);		//set interrupt group priority
	HAL_NVIC_EnableIRQ(TIM3_IRQn);				//Enable the TIMx global interrupt
}

//TIM 인터럽트 Callback 함수 : Period Elapsed callback
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef* htim)
{
	if(htim->Instance == TimHandle2.Instance)
		HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_2);
	else if(htim->Instance == TimHandle3.Instance)
		HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_3);
}

int main(int argc, char* argv[])
{
	//enable GPIOA clock
	__HAL_RCC_GPIOA_CLK_ENABLE();

	//configure PC2 IO in output push-pull mode to driv external LED
	LD1.Pin = GPIO_PIN_2 | GPIO_PIN_3 | GPIO_PIN_4;
	LD1.Mode = GPIO_MODE_OUTPUT_PP;
	LD1.Pull = GPIO_NOPULL;
	LD1.Speed = GPIO_SPEED_LOW;
	HAL_GPIO_Init(GPIOA, &LD1);

	//configure TIMER
	TIMER2_Config();
	TIMER3_Config();

	//메인루틴에서 하는일
  	while (1);
}
```

4. 범용 타이머 실습
딜레이 시간, 작동 시간 나누기

* stm32f4xx_it.h
```
// include--------------------
#include "stm32f4xx_hal.h"

//타이머 인터럽트 처리 핸들러 함수 정의
void TIM2_IRQHandler(void);

```

* stm32f4xx_it.c
```
#include "stm32f4xx_hal.h"
#include "stm32f4xx_it.h"

//TIM인터럽트 ISR을 위한 TimHandle 변수를 외부정의 변수로 선언
extern TIM_HandleTypeDef TimHandle;

//타이머 인터럽트 처리 핸들러 함수 구현
void TIM2_IRQHandler(void)
{
	HAL_TIM_IRQHandler(&TimHandle);
}

```

* main.c
```
#include "stm32f4xx_hal.h"
#include "stm32f4xx_it.h"

TIM_HandleTypeDef TimHandle;		//타이머의 초기화용 구조체 변수를 선언
TIM_OC_InitTypeDef TIM_OCInit;		//타이머 Channel(OC)의 초기화용 구조체 변수를 선언
GPIO_InitTypeDef LED;				//GPIO의 초기화를 위한 구조체형의 변수 선언

//LED의 초기설정용 함수의 선언
void LED_Config()
{
	//led용 GPIO(PA2, PB3)의 초기설정을 함
	__HAL_RCC_GPIOC_CLK_ENABLE();

	LED.Pin = GPIO_PIN_2 | GPIO_PIN_3;		//GPIO에서 사용할 PIN설정
	LED.Mode = GPIO_MODE_OUTPUT_PP;			//output push-pull 모드
	LED.Pull = GPIO_PULLUP;					//Pull up 모드
	LED.Speed = GPIO_SPEED_FREQ_HIGH;		//동작속도를 HIGH로
	HAL_GPIO_Init(GPIOC, &LED);
}

//TIMER의 초기 설정용 함수의 선언
void TIMER_Config()
{
//Tim peripheal lock enable
	__HAL_RCC_TIM2_CLK_ENABLE();

	TimHandle.Instance = TIM2;				//TIM2사용
	TimHandle.Init.Period = 10000-2;		//업데이트 이벤트 발생시 ARR=9999로 설정
	TimHandle.Init.Prescaler = 8400-1;		//Prescaler = 8399로 설정
	TimHandle.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;	//division을 사용하지 않음
	TimHandle.Init.CounterMode = TIM_COUNTERMODE_UP;		//up counter모드 설정
	HAL_TIM_Base_Init(&TimHandle);
	HAL_TIM_Base_Start_IT(&TimHandle);

	//set time oc instance
	TIM_OCInit.OCMode = TIM_OCMODE_TIMING;		//oc동작모드 설정
	TIM_OCInit.Pulse = 8000-1;					//CCR의 설정값
	TIM_OCInit.OCNPolarity = TIM_OCPOLARITY_LOW;		//OC 출력을 Low로 설정
	TIM_OCInit.OCFastMode = TIM_OCFAST_DISABLE;			//TIM output compare fast를  disable
	HAL_TIM_OC_Init(&TimHandle);

	//Tim OC의 Channel을 TIM OCInit에 설정된 값으로 초기화함
	HAL_TIM_OC_ConfigChannel(&TimHandle, &TIM_OCInit, TIM_CHANNEL_1);
	HAL_TIM_OC_Start_IT(&TimHandle, TIM_CHANNEL_1);

	HAL_NVIC_SetPriority(TIM2_IRQn, 0, 0);		//Set interrupt group priority
	HAL_NVIC_EnableIRQ(TIM2_IRQn);				//enable the tim global interrupt

}

//인터럽트 루틴 (callback)

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef* htim)
{
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_2, 1);
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_3, 1);
}
void HAL_TIM_OC_DelayElapsedCallback(TIM_HandleTypeDef* htim)
{
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_2, 0);
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_3, 0);
}

int main(int argc, char* argv[])
{
	//configure LED_1(PA2), LED(PA3)
	LED_Config();

	//Confige TIMER
	TIMER_Config();

  	while (1);

}
```

5. HAl드라이버를 이용한 CHARACTER LCD제어

* main.c

```
#include "stm32f4xx_hal.h"		//관련 레지스터의 주소 지정

GPIO_InitTypeDef LCD;		//GPIO의 초기화를 위한 구조체형의 변수 선언

//지연 루틴
static void ms_delay_int_count(volatile unsigned int nTime)	//ms지연
{
	nTime = (nTime * 14000);
	for(; nTime>0; nTime--);
}
static void us_delay_int_count(volatile unsigned int nTime)	//us지연
{
	nTime = (nTime * 12);
	for(; nTime>0; nTime--);
}

//CLCD의 초기성정용 함수의 선언
void CLCD_Config()
{
	//CLCD용 GPIO의 초기설정을 함
	__HAL_RCC_GPIOC_CLK_ENABLE();

	//CLCD R5(PC8), CLCD_E(PC9, DATA 4~5 (PC12 ~ 15)
	LCD.Pin = GPIO_PIN_8 | GPIO_PIN_9 | GPIO_PIN_12 | GPIO_PIN_13 | GPIO_PIN_14 | GPIO_PIN_15;
	LCD.Mode = GPIO_MODE_OUTPUT_PP;
	LCD.Pull = GPIO_NOPULL;
	LCD.Speed = GPIO_SPEED_FAST;
	HAL_GPIO_Init(GPIOC, &LCD);

	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_9, GPIO_PIN_RESET);		//CLCD_E = 0
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_8, GPIO_PIN_RESET);		//CLCD_RW = 0
}

void CLCD_Write(unsigned char rs, char data)
{
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_8, rs);				//CLDC_RS
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_9, GPIO_PIN_RESET);	//CLDC_E = 0
	us_delay_int_count(2);

	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_12, (data>>4) & 0x1);	//CLDC_DATA = Low_bit
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, (data>>5) & 0x1);
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_14, (data>>6) & 0x1);
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_15, (data>>7) & 0x1);

	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_9, GPIO_PIN_SET);	//CLCD_E = 1
	us_delay_int_count(2);
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_9, GPIO_PIN_RESET);	//CLCD_E = 0
	us_delay_int_count(2);

	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_12, (data>>0) & 0x1);	//CLCD_DATA = HIGH_bit
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, (data>>1) & 0x1);
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_14, (data>>2) & 0x1);
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_15, (data>>3) & 0x1);

	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_9, GPIO_PIN_SET);		//CLCD_E = 1
	us_delay_int_count(2);
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_9, GPIO_PIN_RESET);	//CLCD_E = 0
	ms_delay_int_count(2);
}

void CLCD_Init()
{
	HAL_GPIO_WritePin(GPIOC, GPIO_PIN_9, GPIO_PIN_RESET);		//CLCD_E = 0
	CLCD_Write(0, 0x33);		//4비트 설정 특수 명령
	CLCD_Write(0, 0x32);		//4비트 설정 특수 명령
	CLCD_Write(0, 0x28);		//_set_function
	CLCD_Write(0, 0x0F);		//_set_display
	CLCD_Write(0, 0x01);		//clcd_clear
	CLCD_Write(0, 0x06);		//_set_entry_mode
	CLCD_Write(0, 0x02);		//Return home
}

//메인 루틴
int main(int argc, char* argv[])
{
	//configure CLCD
	CLCD_Config();

	CLCD_Init();

	CLCD_Write(1, 'H');
	CLCD_Write(1, 'e');
	CLCD_Write(1, 'l');
	CLCD_Write(1, 'l');
	CLCD_Write(1, 'o');
	CLCD_Write(1, '!');
	CLCD_Write(1, '!');

	while (1);
}
```

5-1. string을 입력받아 LCD에 출력하기

* main.c
```
void CLCD_Write_String(char* str)
{
	int size = strlen(str);

	for(int i=0; i<size; i++)
	{
		CLCD_Write(1, str[i]);
	}
}
```

해당 함수 추가 후 main에서 
```
CLCD_Write_String("asdfg");
```
다음과 같이 사용


### _initialize_hardware.c 
프로젝트 생성 때 마다 바꿔줘야하는 c파일 
void SystemClock_Config(void) 부분 설정

```
void SystemClock_Config(void)
{

  RCC_OscInitTypeDef RCC_OscInitStruct;
  RCC_ClkInitTypeDef RCC_ClkInitStruct;

  __HAL_RCC_PWR_CLK_ENABLE();

  __HAL_PWR_VOLTAGESCALING_CONFIG(PWR_REGULATOR_VOLTAGE_SCALE1);

  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.HSICalibrationValue = 16;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI;
  RCC_OscInitStruct.PLL.PLLM = 16;
  RCC_OscInitStruct.PLL.PLLN = 336;
  RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV2;
  RCC_OscInitStruct.PLL.PLLQ = 4;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    //Error_Handler();
  }

  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV4;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV2;
  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_5) != HAL_OK)
  {
    //Error_Handler();
  }

  HAL_SYSTICK_Config(HAL_RCC_GetHCLKFreq()/1000);

  HAL_SYSTICK_CLKSourceConfig(SYSTICK_CLKSOURCE_HCLK);

  /* SysTick_IRQn interrupt configuration */
  HAL_NVIC_SetPriority(SysTick_IRQn, 0, 0);
}
```
