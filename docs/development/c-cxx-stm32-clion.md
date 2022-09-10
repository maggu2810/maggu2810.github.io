CLion - STM32
==============

***Development for STM32 using CLion IDE***

**Author:** *Markus Rathgeb*

# Requirements

See also https://www.jetbrains.com/help/clion/embedded-development.html

## OpenOCD

```sh
sudo dnf install \
  openocd
```

## STM32CubeMX

STM32Cube initialization code generator

https://www.st.com/en/development-tools/stm32cubemx.html

* download,  ATM `en.stm32cubemx-lin_v6-6-1.zip`

* unzip
* run setup, ATM `./SetupSTM32CubeMX-6.6.1`
* use a path of your choice e.g. `/home/maggu2810/bin/pkgs/STM32CubeMX`

### CLion

* File
* Settings
* Build, Execution, Deployment
* Embedded Development
* Stm32CubeMX Location: `/home/maggu2810/bin/pkgs/STM32CubeMX/STM32CubeMX`

## ARM Toolchain

```sh
sudo dnf install \
  arm-none-eabi-binutils-cs \
  arm-none-eabi-gcc-cs \
  arm-none-eabi-gcc-cs-c++ \
  arm-none-eabi-newlib
```

# New Project

You could check also https://www.jetbrains.com/help/clion/embedded-development.html

This is my suggestion how to handle it as IMHO the official way does not work as good as it could be.

... TODO ...