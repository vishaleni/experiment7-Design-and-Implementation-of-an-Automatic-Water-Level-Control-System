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
