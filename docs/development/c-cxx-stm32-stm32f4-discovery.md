stm32f4-discovery
==============

***First steps on my first STM32 project***

**Author:** *Markus Rathgeb*

# Create Project - STM32CubeMX

* STM32CubeMX
* Select board
* use default configuration for peripherals
* Project Manager
  * Project Settings
    * Project Name: `stm32-first-steps`
    * Project Location: `/.../project/parent/directory/`
    * Toolchain / IDE: `STM32CubeIDE`
    * Generate Under Root: enabled

* Open "ioc" file in CLion
* Open as Project
* Open with STM32CubeMX
  * Generate Code
  * Close STM32CubeMX
* OpenOCD Dialog
  * Select Board Configuration File
  * `stm32f4discovery.cfg`
  * Use
* New, File, .gitignore: [JetBrains.gitignore](https://github.com/github/gitignore/blob/main/Global/JetBrains.gitignore)

# Update Heap / Stack Size

* Project Manager
  * Linker Settings:
    * Minimum Heap Size: 0x400 (before 0x200)
    * Minimum Stack Size: 0x800 (before 0x400)

# UART printf

## Code

* Open ioc file in STM32CubeMX
  * Connectivity
    * USART2
      * Mode
        * **Mode: Asynchronous**
        * HArdware Flow Control (RS232): Disable
      * Parameter Setting
        * Basic Parameters
          * Baud Rate: 115200 Bits/s
          * Word Length: 8 Bits (including Parity)
          * Parity: None
          * Stop Bits: 1
        * Advanced Parameters
          * Data Direction: Receive and Transmit
          * Over Sampling: 16 Samples
  * Generate Code
* Edit `Core/Src/main.c`

```C
/* USER CODE BEGIN 0 */

int __io_putchar(int ch)
{
  HAL_UART_Transmit(&huart2, (uint8_t *)&ch, 1, 0xFFFF);
  return ch;
}

/* USER CODE END 0 */
```

## Wiring

Connect FTDI / Prolific / ...

| Cable Type | RXD | TXD | GND |
| --------------------- | ------ | ------ | ----- |
| black, yellow, orange | orange | yellow | black |
| black, white, green   | green  | white  | black |

* RXD: PA3
* TXD: PA2
* GND: GND

## Host Side

It seems `screen` and `minicom` do not adding a carriage return on a line feed default. This result into a new line without beginning at the beginning of the line but continue new data on the next position.

For `minicom` we could edit `~/.minirc.dfl` and add the following line

```
pu addcarreturn Yes
```

minicom could be started using (change the device):

```
minicom -b 115200 -D /dev/ttyUSB2 -o
```

## Lecture

See also:
* https://community.st.com/s/question/0D53W00001NvW0ZSAV/stm32g0-redirect-printf-to-write-and-use-uart-how-to
* https://deepbluembedded.com/stm32-debugging-with-uart-serial-print/

# LED Test

```
   <3>
<4>   <5>
   <6>
```

* LED3: orange
* LED4: green
* LED5: red
* LED6: blue

`main.c` main loop (heartbeet)

```C
  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  uint32_t tickLast = HAL_GetTick();
  int next_action = GPIO_PIN_SET;
  while (1)
  {
    /* USER CODE END WHILE */
    MX_USB_HOST_Process();

    /* USER CODE BEGIN 3 */
    const uint32_t tickCur = HAL_GetTick();
    if (tickCur - tickLast >= 1000) {
      tickLast = tickCur;
      HAL_GPIO_WritePin(GPIOD, LD4_Pin, next_action);
      next_action = next_action == GPIO_PIN_SET ? GPIO_PIN_RESET : GPIO_PIN_SET;
    }
  }
  /* USER CODE END 3 */
```

`main.c` error handler

```C
void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */
  __disable_irq();
  uint32_t tickLast = HAL_GetTick();
  int next_action = GPIO_PIN_SET;
  while (1)
  {
    const uint32_t tickCur = HAL_GetTick();
    if (tickCur - tickLast >= 500)
    {
      tickLast = tickCur;
      HAL_GPIO_WritePin(GPIOD, LD6_Pin, next_action);
      next_action = next_action == GPIO_PIN_SET ? GPIO_PIN_RESET : GPIO_PIN_SET;
    }
  }
  /* USER CODE END Error_Handler_Debug */
}
```

# USB Host

## Very import VBUS fix

`USB_HOST/Target/usbh_platform.c`

* on state == 0 change from `GPIO_PIN_RESET` to `GPIO_PIN_SET`
* otherwise (1) change from `GPIO_PIN_SET` to `GPIO_PIN_RESET`

Fixed code looks like:

```C
void MX_DriverVbusFS(uint8_t state)
{
  uint8_t data = state;
  /* USER CODE BEGIN PREPARE_GPIO_DATA_VBUS_FS */
  if(state == 0)
  {
    /* Drive high Charge pump */
    data = GPIO_PIN_SET;
  }
  else
  {
    /* Drive low Charge pump */
    data = GPIO_PIN_RESET;
  }
  /* USER CODE END PREPARE_GPIO_DATA_VBUS_FS */
  HAL_GPIO_WritePin(GPIOC,GPIO_PIN_0,(GPIO_PinState)data);
}
```

For further information see the end of: http://evenlund.blogspot.com/2016/10/usb-storage-with-stm32f4-discovery-and_58.html

# CAN Bus

## Configuration

### Parameter Settings

For the "CAN bit-timing" values' calculation you can use [CAN Bit Time Calculation](http://www.bittiming.can-wiki.info/)

* ST Microelectronics bxCAN
* Clock Rate: 42 MHz
* Sample-Point: 87.5 %
* SJW: 1

STM32 CubeMX Configuration

* Bit Timing Parameters
  * Prescaler (for Time Quantum): 6
  * *Time Quantum: 142.8571... ns*
  * Time Quanta in Bit Segment 1: 11 Times
  * Time Quanta in Bit Segment 2: 2 Time
  * *Time for one Bit: 2000 ns*
  * *Baud Rate: 500000 bit/s*
  * ReSynchronization Jump Width: 1 Time
* Basic Parameters
  * Time Triggered Communication Mode: Disable
  * Automatic Bus-Off Management: Disable
  * Automatic Wake-Up Mode: Disable
  * Automatic Retransmission: Enable
  * Receive Fifo Locked Mode: Disable
  * Transmit Fifo Priority: Disable
* Advanced Parameters
  * Operating Mode: Normal

### GPIO Settings

| PIN Name | Signal on PIN | GPIO output level | GPIO mode                    | GPIO Pull-up/Pull-down | Maximum output speed |
| -------- | ------------- | ----------------- | ---------------------------- | ---------------------- | -------------------- |
| PD0      | CAN1_RX       | n/a               | Alternate Function Push Pull | Pull-up                | Very High            |
| PD1      | CAN1_TX       | n/a               | Alternate Function Push Pull | Pull-up                | Very High            |
