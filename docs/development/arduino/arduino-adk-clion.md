Arduino ADK - CLion
===

***Development for the Arduino ADK using CLion IDE***

**Author:** *Markus Rathgeb*

---

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

## Setup

* Install Plugin `Arduino Support`

## Notes

It seems "third party libraries" are used only if you import the library that name fits to the directory.
