# ESP-IDF Setup

## Get ESP-IDF

```bash
git clone -b v5.1.2 --recursive https://github.com/espressif/esp-idf.git
```

## Install ESP-IDF

```bash
cd esp-idf
./install.sh esp32
```

This script won't work if you are running in a virtual environment (Python) because it will create a new virtual environment and install the required packages in it. So, deactivate the virtual environment before running the script.

```
Installing Python environment and packages
Creating a new Python environment in /Users/alwin/.espressif/python_env/idf5.1_py3.11_env
Downloading https://dl.espressif.com/dl/esp-idf/espidf.constraints.v5.1.txt
Destination: /Users/alwin/.espressif/espidf.constraints.v5.1.txt.tmp
Done
Installing Python packages
 Constraint file: /Users/alwin/.espressif/espidf.constraints.v5.1.txt
 Requirement files:
  - /Users/alwin/explores/qemu-esp32/esp/esp-idf/tools/requirements/requirements.core.txt
Looking in indexes: https://pypi.org/simple, https://dl.espressif.com/pypi
```

### Customizing IDF Tools Path

We can use customized IDF tools path by setting the `IDF_TOOLS_PATH` environment variable before running the `install.sh` script.

```bash
export IDF_TOOLS_PATH=/path/to/custom/tools
./install.sh esp32
```

## Set Up the Environment

```bash
. /path/to/esp-idf/export.sh
```

## Verify the Installation

```bash
idf.py --version
```

It should print the installed version of ESP-IDF.
For example, `v5.1.2`.

```
ESP-IDF v5.1.2
```