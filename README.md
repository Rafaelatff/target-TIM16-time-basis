# target-TIM16-time-basis

NOTE: I tested with semihosting both TIM4 and TIM16 and time basis didn't worked.
Just after removing the semihosting options it started to work.

Configuring to enter the interrupt each 1ms (according with table in [target TIM3 PWM](https://github.com/Rafaelatff/target-TIM3-PWM)). Clock and code condigured by STM32CubeMX.

NOTE: TIM16 is connected to APB2 Timer clock, using 100 MHz as clock.

![image](https://user-images.githubusercontent.com/58916022/227262115-92cc1f5d-a5aa-42bc-8909-2e3c3f2a51fd.png)

```c
void MX_TIM16_Init(void)
{

  /* USER CODE BEGIN TIM16_Init 0 */

  /* USER CODE END TIM16_Init 0 */

  TIM_OC_InitTypeDef sConfigOC = {0};
  TIM_BreakDeadTimeConfigTypeDef sBreakDeadTimeConfig = {0};

  /* USER CODE BEGIN TIM16_Init 1 */

  /* USER CODE END TIM16_Init 1 */
  htim16.Instance = TIM16;
  htim16.Init.Prescaler = 9;
  htim16.Init.CounterMode = TIM_COUNTERMODE_UP;
  htim16.Init.Period = 10000;
  htim16.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
  htim16.Init.RepetitionCounter = 0;
  htim16.Init.AutoReloadPreload = TIM_AUTORELOAD_PRELOAD_DISABLE;
  if (HAL_TIM_Base_Init(&htim16) != HAL_OK)
  {
    Error_Handler();
  }
  if (HAL_TIM_OC_Init(&htim16) != HAL_OK)
  {
    Error_Handler();
  }
  sConfigOC.OCMode = TIM_OCMODE_TIMING;
  sConfigOC.Pulse = 0;
  sConfigOC.OCPolarity = TIM_OCPOLARITY_HIGH;
  sConfigOC.OCNPolarity = TIM_OCNPOLARITY_HIGH;
  sConfigOC.OCFastMode = TIM_OCFAST_DISABLE;
  sConfigOC.OCIdleState = TIM_OCIDLESTATE_RESET;
  sConfigOC.OCNIdleState = TIM_OCNIDLESTATE_RESET;
  if (HAL_TIM_OC_ConfigChannel(&htim16, &sConfigOC, TIM_CHANNEL_1) != HAL_OK)
  {
    Error_Handler();
  }
  sBreakDeadTimeConfig.OffStateRunMode = TIM_OSSR_DISABLE;
  sBreakDeadTimeConfig.OffStateIDLEMode = TIM_OSSI_DISABLE;
  sBreakDeadTimeConfig.LockLevel = TIM_LOCKLEVEL_OFF;
  sBreakDeadTimeConfig.DeadTime = 0;
  sBreakDeadTimeConfig.BreakState = TIM_BREAK_DISABLE;
  sBreakDeadTimeConfig.BreakPolarity = TIM_BREAKPOLARITY_HIGH;
  sBreakDeadTimeConfig.BreakFilter = 0;
  sBreakDeadTimeConfig.AutomaticOutput = TIM_AUTOMATICOUTPUT_DISABLE;
  if (HAL_TIMEx_ConfigBreakDeadTime(&htim16, &sBreakDeadTimeConfig) != HAL_OK)
  {
    Error_Handler();
  }
  /* USER CODE BEGIN TIM16_Init 2 */
  // To start timer by interrupt
  HAL_TIM_Base_Start_IT(&htim16);
  /* USER CODE END TIM16_Init 2 */

}

```

For the callback:

```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *tim_baseHandle){

	if(tim_baseHandle->Instance == TIM16){
		timerMs++;
	}
}
```

And final function with configuration to turn the buzzer on, **x times, y milliseconds on and z milliseconds off**.:

```c
void BuzzerBips(uint16_t times, uint16_t timeOn, uint16_t timeOff){
	// DEBUG Message
	//printf("Beeping %d times, on time = %d ms and off time = %d ms\n", times, timeOn, timeOff);

	uint32_t startValue = timerMs;

	for (uint8_t i=0; i < times ; i++){

		if (HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_4)!= HAL_OK) Error_Handler();
		startValue = timerMs;
		while ((timerMs - startValue) < timeOn);

		if (HAL_TIM_PWM_Stop(&htim3, TIM_CHANNEL_4)!= HAL_OK) Error_Handler();
		startValue = timerMs;
		while ((timerMs - startValue) < timeOff);
	}
}
```
