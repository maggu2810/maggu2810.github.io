Arduino ADK - CLion
===

***Development for the Arduino ADK using CLion IDE***

**Author:** *Markus Rathgeb*

---

# Arduino Mega ADK Information

* https://docs.arduino.cc/retired/boards/arduino-mega-adk-rev3#programming
* https://www.youtube.com/watch?v=h9WIAgpzmRc
* https://www.arduino.cc/en/uploads/Main/arduino-mega-adk-schematic.pdf

# Platform.io

## Setup

See: [CLion, Language and framework-specific guidelines, PlatformIO](https://www.jetbrains.com/help/clion/platformio.html)

## Workaround: Windows Business Certificate

Add your certificate chain to `%HOME%\.platformio\pyenv\Lib\site-packages\certifi\cacert.pem`

### Check

The following seems not be necessary, but need to be checked again: If necessary remove this "introduction", if not
necessary, remove the whole paragraph

* edit `%HOME%\.platformio\pyenv\Lib\site-packages\urllib3\connectionpool.py`
* `class HTTPSConnectionPool`
* `def __init__`
* set `self.ca_certs` to `certifi.where()` (also `import certifi`)

# CLion: Arduino Support Plugin

## One-Time-Setup

* Install Plugin `Arduino Support`
* We need the Arduino SDK.
    * Download the Arduino Legacy IDE (1.8.X) from here [Arduino Software](https://www.arduino.cc/en/software)
    * install

```shell
mkdir ~/bin/pkgs/arduino; tar xJf ~/Downloads/arduino-1.8.19-linux64.tar.xz -C ~/bin/pkgs/arduino
```

## Create Project

* Create new project in CLion
    * New Project
    * `--- Other ---`, `Arduino Sketch`
    * Board: Arduino Mega ADK
    * Programmer: Arduino as ISP
    * Port: /dev/ttyACM0
    * Baud Rate: default

## Find Arduino SDK path

```diff
diff --git a/cmake/ArduinoToolchain.cmake b/cmake/ArduinoToolchain.cmake
index a2891ea..5f877ac 100644
--- a/cmake/ArduinoToolchain.cmake
+++ b/cmake/ArduinoToolchain.cmake
@@ -60,7 +60,8 @@ if (NOT ARDUINO_SDK_PATH)
     endforeach ()
 
     if (UNIX)
-        file(GLOB SDK_PATH_HINTS /usr/share/arduino*
+        file(GLOB SDK_PATH_HINTS $ENV{HOME}/bin/pkgs/arduino/arduino-*
+                /usr/share/arduino*
                 /opt/local/arduino*
                 /opt/arduino*
                 /usr/local/share/arduino*)
```

## Enhance CMake min. Version

```diff
diff --git a/CMakeLists.txt b/CMakeLists.txt
index 763b0f9..70225d3 100644
--- a/CMakeLists.txt
+++ b/CMakeLists.txt
@@ -1,4 +1,4 @@
-cmake_minimum_required(VERSION 3.0.0)
+cmake_minimum_required(VERSION 3.23.0)
set(CMAKE_TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/cmake/ArduinoToolchain.cmake)
set(CMAKE_CXX_STANDARD 17)
set(PROJECT_NAME adk_ac_02)
diff --git a/cmake/Platform/Arduino.cmake b/cmake/Platform/Arduino.cmake
index 6593d12..611d260 100644
--- a/cmake/Platform/Arduino.cmake
+++ b/cmake/Platform/Arduino.cmake
@@ -284,7 +284,7 @@
# License, v. 2.0. If a copy of the MPL was not distributed with this file,
# You can obtain one at http://mozilla.org/MPL/2.0/.
#=============================================================================#
-cmake_minimum_required(VERSION 3.0.0)
+cmake_minimum_required(VERSION 3.23.0)
include(CMakeParseArguments)


```

## Run/Debug Configuration

Run configurations for e.g. upload should be created automatically.

Choose upload and build.

## Notes

It seems "third party libraries" are used only if you import the library that name fits to the directory.

# AOA

## Notes

* [Accessory Development Kit 2012 Guide](https://stuff.mit.edu/afs/sipb/project/android/docs/tools/adk/adk2.html)

## ADK 2012 Source

* http://arduino-er.blogspot.com/2013/03/downloading-adk-2012-source.html

> Sunday, March 3, 2013
>
> # Downloading the ADK 2012 Source
> The [Android Accessory Development Kit (ADK) for 2012](http://developer.android.com/tools/adk/adk2.html) is the latest
> reference implementation of an [Android Open Accessory](http://source.android.com/tech/accessories/index.html) device,
> designed to help Android hardware accessory builders and software developers create accessories for Android. The ADK
> 2012 is based on the [Arduino](http://arduino.cc/) open source electronics prototyping platform, with some hardware and software extensions
> that allow it to communicate with Android devices.
>
> A limited number of these kits were produced and distributed at the Google I/O 2012 developer conference. If you did
> not receive one of these kits, fear not! The specifications and design files for the hardware were also released for use
> by manufacturers and hobbyists. You should expect to see kits with similar features available for purchase, or you can
> build one yourself!
>
> ## Downloading the ADK Source on Ubuntu
>
> The support software and hardware specifications for the ADK 2012 are available from the Android source repository.
> Follow the instructions below to obtain the source material for the ADK.
>
> Before download the source, you have to install curl, git and repo. Refer [here](http://android-er.blogspot.com/2013/03/install-repo-on-ubuntu.html).
>
> Then you can download the ADK 2012 Source, enter the commands in Terminal.
>
> ```shell
> mkdir android-accessories
> cd android-accessories
> repo init -u https://android.googlesource.com/accessories/manifest
> repo sync
> ```
>
> Source: [http://developer.android.com/tools/adk/adk2.html](http://developer.android.com/tools/adk/adk2.html)
>
> After successfully completing this process, you should have the source code and tools for working with the ADK 2012:
>
> * **adk2012/board** - Source code and hardware design files for the ADK 2012
> * **adk2012/app** - Source code for the ADK 2012 Android companion application
> * **external/ide** - Source code for the ADK 2012 Integrated Development Environment (IDE)
> * **external/toolchain** - The toolchain used by the ADK 2012 IDE

## Working Examples

Download UsbHost library with Android Open Accessory support. It can be taken
from [aoabook - Arduino](https://github.com/aoabook/Arduino) in `libraries/UsbHost`.

There you will also find working examples.