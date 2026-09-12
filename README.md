# BME69X and BSEC3.3.0.1 for Python
## Release Note
The bme69x-python-library is a Python 3 wrapper for the BSEC3 library and BME690 environment sensor available from BoschSensortec. 
The main use case for the Raspberry PI is with (1 or 2) single sensor BME690 modules, in IAQ (Air Quality and env data) mode  or SEL mode (Selectivity) sniffing using an AI Studio  model. 

Bosch Sensortec released BSEC v3.3.0.1 in August 2026, with a fix for a missing function in the BSEC3 library for Linux/Raspberry Pi.  This a minor update to the Python3 wrapper for the BSEC 3.3.0.1 release. This release continues to support multiple sensors on the same or multiple I2C buses with isolated configuration and state data. BSEC_OUTPUT_BREATH_VOC_EQUIVALENT was replaced in BSEC3.3.0.0, with BSEC_OUTPUT_TVOC_EQUIVALENT and this continues with BSEC3.3.0.1. TVOC is only reported in LP mode and not in ULP or Scan modes. (See the Bosch Sensortec Integration Guide included with BSEC3.3.0.1).

Also in this release the Python 'build' package is used, rather than calling setup.py directly to build the extension. 

BME690 Sensor modules from Pimoroni are used in development and testing, connected using I2C to Rapberry Pi Zero, Zero2, PI 4, and PI 5. Bosch Sensortec BSEC3.3 supports PI3 ARM v6,  PI3 ARM v8, PI4 ARM v8 (32 and 64bit support).

If you have a BME680 or BME688 please use BSEC2 v2.6.1.0 and the Python wrapper which is stable and has 64bit and 32 bit support [here](https://github.com/mcalisterkm/bme68x-python-library-bsec2.6.1.0).  

## Installation
### Pre-requisites

The BME690 uses I2C which will need to be enabled on the target PI and can be enabled using raspi-config and the "Interface Options" menu.  i2c-tools (apt install i2c-tools) is a useful utility to validate the I2C port your sensor is working on (i2cdetect -y 1). This release supports changing the I2C bus used on the PI, and running multiple sensors (0x76 & 0x77 tested).

The Raspbian Lite OS requires the python3 development package to be installed. (sudo apt install python3-dev). This release uses the Python 'build' package, to build and install this package, which you may not have installed (sudo apt install python3-build).

### How to install the Python module and BSEC3 
The detailed Raspberry Pi installation walkthrough appears below. It covers virtual environments, the licensed Bosch BSEC archive, architecture selection, wheel building, and installation verification.

The high level steps are: 
- setup a Python virtual environment
- clone [this repo](https://github.com/mcalisterkm/bme69x-python-library-bsec3.3.0.1) to a desired location (virtual env) on your hard drive
- download the licensed BSEC3 library [from BOSCH](https://www.bosch-sensortec.com/software-tools/software/bme688-and-bme690-software/)<br>
- unzip it into the *bme69x-python-library-bsec3.3.0.1* folder, next to this *README.md* file
- open a terminal window inside the *bme69x-python-library-bsec3.3.0.1* folder, build a wheel with `python3 -m build`, then install the wheel with `pip`.

Bosch Sensortec provides three binaries in each BSEC3 release: two 32-bit libraries and one 64-bit library. Select the library according to the installed Raspbian OS and the processor architecture, not only the Raspberry Pi model.

| `BSEC` setting | BSEC library | Raspbian OS | Typical Raspberry Pi hardware |
| --- | --- | --- | --- |
| `BSEC=64` | `PiFour_Armv8` | 64-bit | Pi 3B, Pi 4, Pi 5, Pi Zero 2, CM4, CM5 |
| `BSEC=32` | `PiThree_Armv8` | 32-bit | Pi 3B, Pi 4, Pi 5, Pi Zero 2, CM4, CM5 |
| unset | `PiThree_Armv6` | 32-bit | Original Pi Zero and other ArmV6 hardware |

Raspbian 32-bit can run on all of these Raspberry Pi models. Raspbian 64-bit can run on the Pi 3B, Pi 4, Pi 5, Pi Zero 2, CM4, and CM5. The operating system must also match the BSEC library ABI. Check the running environment with `uname -m` and `python3 -c "import platform; print(platform.machine())"`.

Build and install the wheel with the setting that matches your system:

For 64-bit Raspbian on ArmV8 hardware:
```bash
BSEC=64 python3 -m build --wheel
python3 -m pip install dist/*.whl
```

For 32-bit Raspbian on ArmV8 hardware:
```bash
BSEC=32 python3 -m build --wheel
python3 -m pip install dist/*.whl
```

For 32-bit Raspbian on ArmV6 hardware, such as the original Pi Zero, leave `BSEC` unset:
```bash
python3 -m build --wheel
python3 -m pip install dist/*.whl
```

If you change the selected library, remove the previous `build/` and `dist/` directories before rebuilding so that an old wheel is not installed accidentally. A mismatched library can fail during linking with an error such as "file in wrong format". `BSEC3` is accepted as a legacy alias for `BSEC`, but `BSEC` is preferred.

This release supports the Bosch `bsec_v3-3-0-1` archive. For API and workflow guidance, see [Documentation.md](Documentation.md) and [API.md](API.md). Runnable programs are in [`examples/`](examples/).

### How to use the extension
- to import in Python
```python
import bme69x
```
or as a Class
```python
from bme69x import BME69X
```
- See [Documentation.md](Documentation.md) for a quick overview and [API.md](API.md) as a reference.
- to test the installation make sure you connected your BME690 sensor via I2C
- run the following code in a Python3 interpreter
```python
from bme69x import BME69X

# Replace I2C_ADDR with the I2C address of your sensor
# Typically  I2C is  0x76  or 0x77  (Pimoroni BME690 module requires a link to be cut for 0x77)
bme69x = BME69X(I2C_ADDR,1, 0)
bme69x.set_heatr_conf(1, 320, 100, 1)
data = bme69x.get_data()
print(data)
```
## Examples
The examples folder has useful programs, include burning in a sensor, using ultra low power mode, using multiple sensor modules, and  saving and loading config and state data for a sensor. 
## Tools - AI Studio model (sniff)
The tools folder provide a sample AI Model, data and code to use with a BME690 sensor to classify smells. Collecting data is best done with the BME690  8 sensor BOSCH Sensortec DevKit


## Installation walk through with Raspbian trixie 64bit on a  PI4.
```
$ sudu apt install i2c-tools
$ i2cdetect -y 1
0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- 77 
```
The BME690 module is showing up on port 0x77.
If it fails to show up check connections and the module documentation. 

Install "Python3-dev" and ""Python3-build" packages (If you miss this step you may see a Python.h missing error or a build failure)
```
$ sudo apt install python3-dev
$ sudo apt install python3-build
```

As this system uses Raspbian trixe, a virtual environment is required (also required for Bookworm).
```
$ python -m venv --system-site-packages BSEC3.3.0.1
$ cd BSEC3.3.0.1
$ source bin/activate
<user>:~/BSEC3.3.0.1 $
```
Note: If you need to run the venv python3 from init scripts, cron, or systemd timers (anything that runs outside of the venv), provide the full path to python3 in the venv bin folder. It is that simple!

Next clone this repo into the virtual environment.
```
(690)<user>:~/BSEC3.3.0.1 $ ls -l
total 20
drwxr-xr-x  2 kpi kpi 4096 Jun 23 23:22 bin
drwxr-xr-x 11 kpi kpi 4096 Jun 23 23:32 bme69x-python-library-bsec3.3.0.1
drwxr-xr-x  3 kpi kpi 4096 Jun 23 23:22 include
drwxr-xr-x  3 kpi kpi 4096 Jun 23 23:22 lib
-rw-r--r--  1 kpi kpi  173 Jun 23 23:22 pyvenv.cfg
```

Now copy the Bosch Sensortec bsec_v3-3-0-1 directory into the bme69x repo clone.
It should look like this.

```
$ ls -l
-rw-r--r-- 1 kpi kpi  9563 Nov 30 19:18 API.md
drwxr-xr-x 4 kpi kpi  4096 Apr 13  2025 BME690_SensorAPI
-rw-r--r-- 1 kpi kpi  2933 Apr 13  2025 bme69xConstants.py
-rw-r--r-- 1 kpi kpi 85683 Nov 30 23:55 bme69xmodule.c
-rw-r--r-- 1 kpi kpi   571 Apr 13  2025 bsecConstants.py
drwxr-xr-x 5 kpi kpi  4096 Nov 30 19:11 bsec_v3-3-0-1
drwxr-xr-x 5 kpi kpi  4096 Jul  6 18:17 build
-rw-r--r-- 1 kpi kpi  7835 Nov 30 08:32 Documentation.md
drwxr-xr-x 4 kpi kpi  4096 Nov 30 23:57 examples
-rw-r--r-- 1 kpi kpi 17625 Nov 30 23:55 internal_functions.c
-rw-r--r-- 1 kpi kpi  2490 Nov 30 23:54 internal_functions.h
-rw-r--r-- 1 kpi kpi  1065 Apr 13  2025 LICENSE
-rw-r--r-- 1 kpi kpi 10249 Aug 12 15:41 README.md
-rw-r--r-- 1 kpi kpi  3422 Nov 30 22:41 setup.py

```
From here build the wheel with the 64bit env set for a PI4 board with 64 bit raspbian, then install it into the active venv. Note: As discussed above the environment variable BSEC needs to be one of 64, 32, or not set. 

```
(BSEC3.3.0.1) <user>:~/BSEC3.3.0.1/bme69x-python-library-bsec3.3.0.1 $ BSEC=64  python3 -m build --wheel
* Creating isolated environment: venv+pip...
* Installing packages in isolated environment:
  - setuptools>=68
  - wheel
* Getting build dependencies for wheel...
running egg_info
creating bme69x.egg-info
.......
.........
adding 'bme69x-3.3.0.1.dist-info/top_level.txt'
adding 'bme69x-3.3.0.1.dist-info/RECORD'
removing build/bdist.linux-aarch64/wheel
Successfully built bme69x-3.3.0.1-cp313-cp313-linux_aarch64.whl


(BSEC3.3.0.1) kpi@PI43:~/BSEC3.3.0.1/bme69x-python-library-bsec3.3.0.1 $ python3 -m pip install dist/*.whl
Looking in indexes: https://pypi.org/simple, https://www.piwheels.org/simple
Processing ./dist/bme69x-3.3.0.1-cp313-cp313-linux_aarch64.whl
Installing collected packages: bme69x
Successfully installed bme69x-3.3.0.1

```

The build should be clean with no warnings, and the wheel build should complete with output under the `dist/` directory.

After installing the wheel, verify it with:
```
(BSEC3.3.0.1) kpi@PI43:~/BSEC3.3.0.1/bme69x-python-library-bsec3.3.0.1 $ python3 -m pip show bme69x
Name: bme69x
Version: 3.3.0.1
Summary: pi3g Python interface for BME69X sensor and BSEC

```

Change to the examples directory and run the forced mode example:
```
(BSEC3.3.0.1) <user>:~/BSEC3.3.0.1/bme69x-python-library-bsec3.3.0.1/examples $ ls
airquality.py  build       conf            force_ulp.py          parallel_mode.py      read_conf.py
bme_ptrs.log   burn_in.py  forced_mode.py  multi_sensor_test.py  parallel_mode_ulp.py  README.md

(BSEC3.3.0.1) <user>:~/BSEC3.3.0.1/bme69x-python-library-bsec3.3.0.1/examples $ python3 forced_mode.py
TESTING FORCED MODE WITHOUT BSEC
{'sample_nr': 1, 'timestamp': 3956208, 'raw_temperature': 54.29930877685547, 'raw_pressure': 931.8998413085938, 'raw_humidity': 108.98556518554688, 'raw_gas': 109.448486328125, 'status': 160}

TESTING FORCED MODE WITH BSEC
{'sample_nr': 1, 'timestamp': 626732524746608, 'iaq': 50.0, 'iaq_accuracy': 0, 'static_iaq': 50.0, 'static_iaq_accuracy': 0, 'co2_equivalent': 500.0, 'co2_accuracy': 0, 'raw_temperature': 20.232421875, 'raw_pressure': 100879.46875, 'raw_humidity': 44.31648254394531, 'raw_gas': 25241.5703125, 'stabilization_status': 128, 'run_in_status': 144, 'temperature': 15.232421875, 'humidity': 60.71345901489258, 'gas_percentage': 0.0, 'gas_percentage_accuracy': 0, 'tvoc_equivalent': 0.0, 'tvoc_equivalent_accuracy': 0}
```
As the status and accuracy are all zero it is time to burn in this sensor for 24 hours. 

The original PI3G repository is available [here] (https://github.com/pi3g/bme68x-python-library) which works with BSEC 2.0.6.1/BME68x (32bit) from 2022.

