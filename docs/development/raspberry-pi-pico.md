Raspberry Pi Pico
===

***... related information***

**Author**: *Markus Rathgeb*

---

# Projects

* [Raspberry Pi Pico USB I/O Board](https://github.com/notro/pico-usb-io-board)

# Setup

* Let's install all Pico related open source stuff to: `${HOME}/workspace/oss/pico`
* We assume you want to develop on a PC and use Picoprobe / Debugprobe

## Pinout

[https://datasheets.raspberrypi.com/pico/Pico-R3-A4-Pinout.pdf](https://datasheets.raspberrypi.com/pico/Pico-R3-A4-Pinout.pdf)

## Environment File

Create an environment file: `~/bin/env-pico`

```
export PICO_BASEDIR="${HOME}"/workspace/oss/pico
export PICO_SDK_PATH="${PICO_BASEDIR}"/pico-sdk
```

## SDK

### Setup

```
. ~/bin/env-pico
git clone -b master https://github.com/raspberrypi/pico-sdk.git "${PICO_SDK_PATH}"
cd "${PICO_SDK_PATH}"
git submodule update --init
```

### Update

```
. ~/bin/env-pico
cd "${PICO_SDK_PATH}"
git pull
git submodule update
```

## Examples

```
. ~/bin/env-pico
git clone -b master https://github.com/raspberrypi/pico-examples.git "${PICO_BASEDIR}"/pico-examples
```

## Toolchain

Ubuntu

```
apt install \
  cmake \
  gcc-arm-none-eabi \
  libnewlib-arm-none-eabi \
  build-essential
```

Fedora

```
sudo dnf install \
  arm-none-eabi-gcc-cs \
  arm-none-eabi-gcc-cs-c++ \
  arm-none-eabi-newlib \
  arm-none-eabi-binutils-cs \
  cmake
```

## debugprobe

[https://github.com/raspberrypi/debugprobe/releases](https://github.com/raspberrypi/debugprobe/releases)

![](https://community.platformio.org/uploads/default/optimized/3X/4/6/46c013bfa5b8e5eaeccb4f48d68851ab5046c891_2_511x500.png)

## OpenOCD

[https://github.com/raspberrypi/openocd](https://github.com/raspberrypi/openocd)

### Dependencies

Ubuntu

```
apt install automake autoconf build-essential texinfo libtool libftdi-dev libusb-1.0-0-dev
```

Fedora

```
sudo dnf install libtool libusb1-devel
```

### Checkout

```
git clone https://github.com/raspberrypi/openocd.git --branch picoprobe --depth=1 --no-single-branch "${PICO_BASEDIR}"/openocd
```

### Build and Install

```
cd "${PICO_BASEDIR}"/openocd

./bootstrap

export CFLAGS="-Wno-error=misleading-indentation -Wno-error=stringop-overflow -Wno-error=calloc-transposed-args"
export CXXFLAGS="-Wno-error=misleading-indentation -Wno-error=stringop-overflow -Wno-error=calloc-transposed-args"

./configure \
  --enable-picoprobe \
  --prefix="${PICO_BASEDIR}"/openocd.install

make -j$(nproc)

unset CFLAGS
unset CXXFLAGS

make install

sudo cp -v ${PICO_BASEDIR}/openocd.install/share/openocd/contrib/60-openocd.rules /etc/udev/rules.d
sudo udevadm control --reload

cat <<EOF >"${PICO_BASEDIR}"/openocd.install/share/openocd/scripts/board/debugprobe_picoprobe_rp2040.cfg
source [find interface/picoprobe.cfg]
source [find target/rp2040.cfg]
EOF

cat <<EOF >"${PICO_BASEDIR}"/openocd.install/share/openocd/scripts/board/debugprobe_cmsis-dap_picoprobe_rp2040.cfg
source [find interface/cmsis-dap.cfg]
source [find target/rp2040.cfg]
# For the cmsis-dap part the adapter speed must be provided, othewise you will get "Error: CMSIS-DAP command CMD_DAP_SWJ_CLOCK failed."
# See: https://github.com/raspberrypi/debugprobe/issues/48#issuecomment-1336719133
adapter speed 5000
EOF
```

### Interface

Depending on the version of the debugprobe uf2 implementation you have to use picoprobe (2e8a:0004) or cmsis-dap (2e8a:000c).

### Use on command line

```
"${PICO_BASEDIR}"/openocd.install/bin/openocd \
  -f interface/picoprobe.cfg \
  -f target/rp2040.cfg
  -s tcl
```

For the cmsis-dap part the adapter speed must be provided, othewise you will get "Error: CMSIS-DAP command CMD_DAP_SWJ_CLOCK failed."

See: [https://github.com/raspberrypi/debugprobe/issues/48#issuecomment-1336719133](https://github.com/raspberrypi/debugprobe/issues/48#issuecomment-1336719133)

```
"${PICO_BASEDIR}"/openocd.install/bin/openocd \
  -f interface/cmsis-dap.cfg \
  -f target/rp2040.cfg \
  -c 'adapter speed 5000' \
  -s tcl
```

```
"${PICO_BASEDIR}"/openocd.install/bin/openocd \
  -f board/debugprobe_cmsis-dap_picoprobe_rp2040.cfg \
  -s tcl
```

Reset

```
"${PICO_BASEDIR}"/openocd.install/bin/openocd \
  -f board/debugprobe_cmsis-dap_picoprobe_rp2040.cfg \
  -s tcl \
  -c "init; reset; exit"
```

Upload ELF

```
"${PICO_BASEDIR}"/openocd.install/bin/openocd \
  -f board/debugprobe_cmsis-dap_picoprobe_rp2040.cfg \
  -s tcl \
  -c "targets rp2040.core0; program $1 preverify verify reset exit"
```

## CLion

CLion

- File, Settings
    - Build, Execution, Development
        - Toolchains
            - +, System
                - Name: Pico
                - Add Environment, From file
                - Environment file: /home/.../bin/env-pico
            - Apply
        - Embedded Development
            - OpenOCD Location: /home/.../workspace/oss/pico/openocd.install/bin/openocd

  

Open CMake project and choose Toolchain Pico (and for the moment Unix Makefiles – Ninja currently not installed and so not tested).

  

Example for a "Run / Debug Configurations"

- +
    - OpenOCD Download & Run
        - Name: OpenOCD blink
        - Target: blink
        - Executable: blink
        - Board config file: board/debugprobe_cmsis-dap_picoprobe_rp2040.cfg
        - GDB port: 3333
        - Telnet port: 4444
        - Download: Always
        - Reset: Init

Use always "Debug" and NOT "Run".

## CMake Project Template

```
cmake_minimum_required(VERSION 3.21)

# Pull in SDK (must be before project)
include($ENV{PICO_SDK_PATH}/pico_sdk_init.cmake)

project(buzzer C CXX ASM)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

# Seems to be necessary for debugging
set(PICO_DEOPTIMIZED_DEBUG 1)

# Initialize the SDK
pico_sdk_init()

add_compile_options(
  -Wall`
  -Wno-format          # int != int32_t as far as the compiler is concerned because gcc has int32_t as long int
  -Wno-unused-function # we have some for the docs that aren't called
  -Wno-maybe-uninitialized
)

add_executable(buzzer
  src/main.cxx
)
target_include_directories(buzzer PRIVATE src)

# pull in common dependencies
target_link_libraries(buzzer pico_stdlib)

# create map/bin/hex file etc.
pico_add_extra_outputs(buzzer)
```
  

# OLD

## Project generator

```bash
cd "${PICO_BASEDIR}"
git clone https://github.com/raspberrypi/pico-project-generator.git
cd pico-project-generator
```

GUI:

```
cd "${PICO_BASEDIR}"/pico-project-generator
./pico_project.py --gui
```

Command line:

```bash
export PICO_SDK_PATH="/home/pi/pico/pico-sdk"
./pico_project.py --feature spi --feature i2c --project vscode test
```

## SVD

```
${env:PICO_SDK_PATH}/src/rp2040/hardware_regs/rp2040.svd
```

## Picotool

```bash
sudo apt-get install libusb-1.0-0-dev
```

```bash
cd "${PICO_BASEDIR}"
git clone -b master https://github.com/raspberrypi/picotool.git
cd picotool
cmake -S . -B build
make -C build
```

`"${PICO_BASEDIR}"/picotool/build/picotool`