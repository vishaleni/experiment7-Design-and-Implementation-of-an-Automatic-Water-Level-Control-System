# Experiment: Design and Implementation of a Water Level Indicator Using STM32

## Aim

To design and implement a **Water Level Indicator using STM32 Nucleo-L031K6** that measures the water level using an analog input and displays the water level as **LOW, MEDIUM, or HIGH** on the Wokwi Serial Monitor.

---

## Components Required

- STM32 Nucleo-L031K6
- Potentiometer to simulate the water-level sensor
- Wokwi Simulator
- Virtual Serial Monitor
- Connecting wires

---

## Theory

A **Water Level Indicator** is used to monitor the amount of water present in a tank.

In this experiment, a **potentiometer** is used in Wokwi to simulate the output of a water-level sensor. The potentiometer produces an analog voltage between **0 V and 3.3 V**.

The analog signal is connected to the **PA0 ADC input** of the STM32 Nucleo-L031K6.

The STM32 contains a **12-bit Analog-to-Digital Converter (ADC)**. Therefore, the analog input voltage is converted into a digital value between:

- **0** → Minimum water level
- **4095** → Maximum water level

The ADC value is converted into a percentage, and the water level is classified as:

- **LOW**
- **MEDIUM**
- **HIGH**

The water-level information is displayed on the **Wokwi Serial Monitor** using UART communication.

---

## Pin Configuration

| Component | STM32 Pin | Function |
|---|---|---|
| Potentiometer SIG | PA0 | ADC input |
| Potentiometer VCC | 3.3V | Power supply |
| Potentiometer GND | GND | Ground |
| USART2 TX | PA2 | Serial data transmission |
| USART2 RX | PA15 | Serial data reception |

---

## Block Diagram

~~~text
       Potentiometer
   (Water Level Sensor)
          │
          │ Analog Signal
          ▼
       PA0 / ADC
          │
          ▼
+-----------------------+
| STM32 Nucleo-L031K6   |
|                       |
| ADC Conversion        |
|        ↓              |
| Water Level %         |
|        ↓              |
| LOW / MEDIUM / HIGH   |
+-----------+-----------+
            │
            │ USART2
            ▼
      Serial Monitor
            │
            ▼
   Water Level Display
~~~

---

## Water Level Classification

| Water Level | Percentage | Indication |
|---|---:|---|
| Low | 0–30% | LOW |
| Medium | 31–70% | MEDIUM |
| High | 71–100% | HIGH |

---

## Algorithm

1. Start the program.
2. Initialize the STM32 HAL library.
3. Configure the system clock.
4. Configure **PA0 as an analog input**.
5. Initialize ADC1.
6. Initialize USART2 for serial communication.
7. Read the analog value from the potentiometer.
8. Convert the ADC reading into water-level percentage.
9. Compare the percentage with predefined limits.
10. If the water level is less than or equal to 30%, display **LOW**.
11. If the water level is between 31% and 70%, display **MEDIUM**.
12. If the water level is greater than 70%, display **HIGH**.
13. Display the ADC value, water-level percentage, and status on the Serial Monitor.
14. Wait for one second.
15. Repeat continuously.

---

## Program
```

#include <stdio.h>
#include <stdint.h>
#include <stm32l0xx_hal.h>

/* Built-in LED representing the water pump */
#define PUMP_LED_PORT              GPIOB
#define PUMP_LED_PIN               GPIO_PIN_3
#define PUMP_LED_CLK_ENABLE()      __HAL_RCC_GPIOB_CLK_ENABLE()

/* Soil-moisture sensor analog input */
#define SOIL_SENSOR_PORT           GPIOA
#define SOIL_SENSOR_PIN            GPIO_PIN_0
#define SOIL_SENSOR_CHANNEL        ADC_CHANNEL_0

/* USART2 virtual COM port pins */
#define VCP_TX_PIN                 GPIO_PIN_2
#define VCP_RX_PIN                 GPIO_PIN_15

/*
 * Higher ADC value = dry soil
 * Lower ADC value  = wet soil
 *
 * Separate thresholds provide hysteresis and prevent
 * repeated ON/OFF switching near one threshold.
 */
#define PUMP_ON_THRESHOLD          2800
#define PUMP_OFF_THRESHOLD         2200

UART_HandleTypeDef huart2;
ADC_HandleTypeDef hadc1;

void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_ADC1_Init(void);
static void MX_USART2_UART_Init(void);
static uint32_t Read_Soil_Moisture(void);
void Error_Handler(void);

int main(void)
{
  uint32_t soilValue;
  uint32_t moisturePercentage;
  uint8_t pumpStatus = 0;

  HAL_Init();
  SystemClock_Config();

  MX_GPIO_Init();
  MX_ADC1_Init();
  MX_USART2_UART_Init();

  printf("\r\n====================================\r\n");
  printf("STM32 Smart Irrigation System\r\n");
  printf("====================================\r\n");
  printf("PA0 : Soil-moisture sensor\r\n");
  printf("PB3 : Pump indicator LED\r\n\r\n");

  /*
   * Calibrate the ADC before taking readings.
   */
  if (HAL_ADCEx_Calibration_Start(&hadc1, ADC_SINGLE_ENDED) != HAL_OK)
  {
    Error_Handler();
  }

  while (1)
  {
    soilValue = Read_Soil_Moisture();

    /*
     * Convert ADC value into moisture percentage.
     *
     * ADC = 0    means approximately 100% wet.
     * ADC = 4095 means approximately 0% wet.
     */
    moisturePercentage =
        100U - ((soilValue * 100U) / 4095U);

    /*
     * Turn ON the pump when the soil becomes dry.
     */
    if ((soilValue >= PUMP_ON_THRESHOLD) && (pumpStatus == 0))
    {
      pumpStatus = 1;

      HAL_GPIO_WritePin(
          PUMP_LED_PORT,
          PUMP_LED_PIN,
          GPIO_PIN_SET);

      printf("Soil is dry: Pump switched ON\r\n");
    }

    /*
     * Turn OFF the pump when sufficient moisture is reached.
     */
    else if ((soilValue <= PUMP_OFF_THRESHOLD) && (pumpStatus == 1))
    {
      pumpStatus = 0;

      HAL_GPIO_WritePin(
          PUMP_LED_PORT,
          PUMP_LED_PIN,
          GPIO_PIN_RESET);

      printf("Soil is wet: Pump switched OFF\r\n");
    }

    printf("ADC value: %lu | Moisture: %lu%% | Pump: %s\r\n",
           (unsigned long)soilValue,
           (unsigned long)moisturePercentage,
           pumpStatus ? "ON" : "OFF");

    printf("------------------------------------\r\n");

    HAL_Delay(1000);
  }
}

/**
 * Read the soil-moisture sensor through ADC1.
 */
static uint32_t Read_Soil_Moisture(void)
{
  uint32_t adcValue;

  if (HAL_ADC_Start(&hadc1) != HAL_OK)
  {
    Error_Handler();
  }

  if (HAL_ADC_PollForConversion(&hadc1, 100) != HAL_OK)
  {
    Error_Handler();
  }

  adcValue = HAL_ADC_GetValue(&hadc1);

  HAL_ADC_Stop(&hadc1);

  return adcValue;
}

/**
 * GPIO initialization.
 */
static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  PUMP_LED_CLK_ENABLE();
  __HAL_RCC_GPIOA_CLK_ENABLE();

  /*
   * Configure PB3 as the pump indicator output.
   */
  GPIO_InitStruct.Pin = PUMP_LED_PIN;
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;

  HAL_GPIO_Init(PUMP_LED_PORT, &GPIO_InitStruct);

  /*
   * Initially keep the pump OFF.
   */
  HAL_GPIO_WritePin(
      PUMP_LED_PORT,
      PUMP_LED_PIN,
      GPIO_PIN_RESET);

  /*
   * Configure PA0 as an analog input.
   */
  GPIO_InitStruct.Pin = SOIL_SENSOR_PIN;
  GPIO_InitStruct.Mode = GPIO_MODE_ANALOG;
  GPIO_InitStruct.Pull = GPIO_NOPULL;

  HAL_GPIO_Init(SOIL_SENSOR_PORT, &GPIO_InitStruct);
}

/**
 * ADC1 initialization.
 */
static void MX_ADC1_Init(void)
{
  ADC_ChannelConfTypeDef channelConfig = {0};

  __HAL_RCC_ADC1_CLK_ENABLE();

  hadc1.Instance = ADC1;
  hadc1.Init.OversamplingMode = DISABLE;
  hadc1.Init.ClockPrescaler = ADC_CLOCK_SYNC_PCLK_DIV2;
  hadc1.Init.Resolution = ADC_RESOLUTION_12B;
  hadc1.Init.SamplingTime = ADC_SAMPLETIME_39CYCLES_5;
  hadc1.Init.ScanConvMode = ADC_SCAN_DIRECTION_FORWARD;
  hadc1.Init.DataAlign = ADC_DATAALIGN_RIGHT;
  hadc1.Init.ContinuousConvMode = DISABLE;
  hadc1.Init.DiscontinuousConvMode = DISABLE;
  hadc1.Init.ExternalTrigConvEdge = ADC_EXTERNALTRIGCONVEDGE_NONE;
  hadc1.Init.ExternalTrigConv = ADC_SOFTWARE_START;
  hadc1.Init.DMAContinuousRequests = DISABLE;
  hadc1.Init.EOCSelection = ADC_EOC_SINGLE_CONV;
  hadc1.Init.Overrun = ADC_OVR_DATA_PRESERVED;
  hadc1.Init.LowPowerAutoWait = DISABLE;
  hadc1.Init.LowPowerFrequencyMode = DISABLE;
  hadc1.Init.LowPowerAutoPowerOff = DISABLE;

  if (HAL_ADC_Init(&hadc1) != HAL_OK)
  {
    Error_Handler();
  }

  channelConfig.Channel = SOIL_SENSOR_CHANNEL;
  channelConfig.Rank = ADC_RANK_CHANNEL_NUMBER;

  if (HAL_ADC_ConfigChannel(&hadc1, &channelConfig) != HAL_OK)
  {
    Error_Handler();
  }
}

/**
 * System clock configuration.
 */
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};
  RCC_PeriphCLKInitTypeDef PeriphClkInit = {0};

  __HAL_PWR_VOLTAGESCALING_CONFIG(
      PWR_REGULATOR_VOLTAGE_SCALE1);

  RCC_OscInitStruct.OscillatorType =
      RCC_OSCILLATORTYPE_HSI;

  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.HSICalibrationValue =
      RCC_HSICALIBRATION_DEFAULT;

  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI;
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLLMUL_4;
  RCC_OscInitStruct.PLL.PLLDIV = RCC_PLLDIV_2;

  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  RCC_ClkInitStruct.ClockType =
      RCC_CLOCKTYPE_HCLK |
      RCC_CLOCKTYPE_SYSCLK |
      RCC_CLOCKTYPE_PCLK1 |
      RCC_CLOCKTYPE_PCLK2;

  RCC_ClkInitStruct.SYSCLKSource =
      RCC_SYSCLKSOURCE_PLLCLK;

  RCC_ClkInitStruct.AHBCLKDivider =
      RCC_SYSCLK_DIV1;

  RCC_ClkInitStruct.APB1CLKDivider =
      RCC_HCLK_DIV1;

  RCC_ClkInitStruct.APB2CLKDivider =
      RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(
          &RCC_ClkInitStruct,
          FLASH_LATENCY_1) != HAL_OK)
  {
    Error_Handler();
  }

  PeriphClkInit.PeriphClockSelection =
      RCC_PERIPHCLK_USART2;

  PeriphClkInit.Usart2ClockSelection =
      RCC_USART2CLKSOURCE_PCLK1;

  if (HAL_RCCEx_PeriphCLKConfig(
          &PeriphClkInit) != HAL_OK)
  {
    Error_Handler();
  }
}

/**
 * USART2 initialization.
 */
static void MX_USART2_UART_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOA_CLK_ENABLE();
  __HAL_RCC_USART2_CLK_ENABLE();

  /*
   * PA2  -> USART2_TX
   * PA15 -> USART2_RX
   */
  GPIO_InitStruct.Pin = VCP_TX_PIN | VCP_RX_PIN;
  GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_VERY_HIGH;
  GPIO_InitStruct.Alternate = GPIO_AF4_USART2;

  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

  huart2.Instance = USART2;
  huart2.Init.BaudRate = 115200;
  huart2.Init.WordLength = UART_WORDLENGTH_8B;
  huart2.Init.StopBits = UART_STOPBITS_1;
  huart2.Init.Parity = UART_PARITY_NONE;
  huart2.Init.Mode = UART_MODE_TX_RX;
  huart2.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart2.Init.OverSampling = UART_OVERSAMPLING_16;
  huart2.Init.OneBitSampling =
      UART_ONE_BIT_SAMPLE_DISABLE;

  huart2.AdvancedInit.AdvFeatureInit =
      UART_ADVFEATURE_NO_INIT;

  if (HAL_UART_Init(&huart2) != HAL_OK)
  {
    Error_Handler();
  }
}

/**
 * Error handler.
 */
void Error_Handler(void)
{
  HAL_GPIO_WritePin(
      PUMP_LED_PORT,
      PUMP_LED_PIN,
      GPIO_PIN_RESET);

  while (1)
  {
    /* Remain here when an initialization error occurs. */
  }
}

/*
 * Redirect printf() output to USART2.
 */
#define STDOUT_FILENO 1
#define STDERR_FILENO 2

int _write(int file, uint8_t *ptr, int len)
{
  if ((file == STDOUT_FILENO) ||
      (file == STDERR_FILENO))
  {
    HAL_UART_Transmit(
        &huart2,
        ptr,
        len,
        HAL_MAX_DELAY);

    return len;
  }

  return -1;
```
} 


## Circuit Connections
<img width="505" height="277" alt="image" src="https://github.com/user-attachments/assets/190f436e-eeab-4f1c-b26b-b7f221a14441" />
### Potentiometer
              Potentiometer
         (Water Level Sensor)
        +--------------------+
3.3V ---| VCC                |
GND  ---| GND                |
        | SIG                 |
        +--+-----------------+
           |
           |
          PA0
           |
           v
+---------------------------+
| STM32 Nucleo-L031K6       |
|                           |
| PA0  -> ADC Input         |
|                           |
| PA2  -> USART2 TX         |
| PA15 -> USART2 RX         |
+-------------+-------------+
              |
              | UART
              v
       Wokwi Serial Monitor

| Potentiometer Pin | STM32 Connection |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SIG / Middle Pin | PA0 |

---

## Circuit Diagram

~~~text
              Potentiometer
         (Water Level Sensor)
        +--------------------+
3.3V ---| VCC                |
GND  ---| GND                |
        | SIG                 |
        +--+-----------------+
           |
           |
          PA0
           |
           v
+---------------------------+
| STM32 Nucleo-L031K6       |
|                           |
| PA0  -> ADC Input         |
|                           |
| PA2  -> USART2 TX         |
| PA15 -> USART2 RX         |
+-------------+-------------+
              |
              | UART
              v
       Wokwi Serial Monitor
~~~

---

## Procedure

1. Open **Wokwi**.
2. Select the **STM32 Nucleo-L031K6** board.
3. Add a **potentiometer**.
4. Connect the potentiometer VCC pin to **3.3V**.
5. Connect the potentiometer GND pin to **GND**.
6. Connect the potentiometer SIG pin to **PA0**.
7. Enter the STM32 HAL program.
8. Compile the program.
9. Start the simulation.
10. Open the **Serial Monitor**.
11. Rotate the potentiometer to simulate different water levels.
12. Observe the ADC value, water-level percentage, and water-level status.

---

## Expected Output

### Low Water Level

~~~text
ADC Value: 800
Water Level: 19%
Status: LOW
~~~

### Medium Water Level

~~~text
ADC Value: 2200
Water Level: 53%
Status: MEDIUM
~~~

### High Water Level

~~~text
ADC Value: 3500
Water Level: 85%
Status: HIGH
~~~

---

## Working

The potentiometer produces an analog voltage that represents the water level in the tank.

The STM32 reads this analog voltage through the **PA0 ADC input**. Since the ADC has a **12-bit resolution**, the analog signal is converted into a digital value between **0 and 4095**.

The ADC value is converted into water-level percentage using:

`Water Level (%) = (ADC Value × 100) / 4095`

The STM32 then classifies the water level as:

- **0–30% → LOW**
- **31–70% → MEDIUM**
- **71–100% → HIGH**

The ADC value, percentage, and water-level status are transmitted through **USART2** and displayed on the **Wokwi Serial Monitor**.

---

## Result

Thus, the **Water Level Indicator using STM32 Nucleo-L031K6** was designed and implemented successfully. The water level was measured using the ADC and displayed as **LOW, MEDIUM, or HIGH** on the Wokwi Serial Monitor.
