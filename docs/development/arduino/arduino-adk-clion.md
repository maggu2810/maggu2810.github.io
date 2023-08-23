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

The following seems not be necessary, but need to be checked again: If necessary remove this "introduction", if not necessary, remove the whole paragraph

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
