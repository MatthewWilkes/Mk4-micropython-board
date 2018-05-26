# TI SimpleLink TI-RTOS MicroPython Port

## Build

* Linux only

* Boards
  * MSP_EXP432E401Y
  * MSP_EXP432E401Y + CC3120 BP (dual network)
  * MSP_EXP432P401R
  * CC3220SF_LAUNCHXL
  * CC1352R1_LAUNCHXL

YOUR_BOARD below refers to one of the above

### SDKs and Tools

To build a MicroPython (MP) library and executable, tools and SDKs are needed:

* GCC ARM compiler
* SDKs - drivers and stacks and kernels for each device/SoC family

Two options:

Use the `inst_tools` script to download the requirements

``` shell
.../ports/ti% ./inst_tools YOUR_BOARD
```

Note: the CC3220 and CC13xx and CC26xx SDKs cannot be downloaded anonymously so you will need to use the links below to manually download and then install.

Manually download and install

* Download and install the SimpleLink SDK for the desired device family, 2.10+
  * MSP432E4: http://www.ti.com/tool/download/SIMPLELINK-MSP432E4-SDK
  * CC3220: http://www.ti.com/tool/SIMPLELINK-CC3220-SDK
  * MSP432(P4): http://www.ti.com/tool/SIMPLELINK-MSP432-SDK
  * CC13xx: http://www.ti.com/tool/simplelink-cc13x2-sdk (out-of-date: using internal Core SDK 1Q18 release for now)

* Download and install GCC toolchain: https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads

See the `inst_tools` file for required versions to download.

``` shell
.../ports/ti% mkdir tools
.../ports/ti/tools% # install gcc and SDK(s)
```

### Make

After installing the SDK and GCC tools:

* MicroPython
  * git clone https://github.com/micropython/micropython.git
  * cd micropython
  * git submodule add `git-url:mp_ports_ti` # ask for actual URL
  * cd ports/ti
  * make BOARD=YOUR_BOARD

The result should be:

* a library in build-YOUR_BOARD/libmicropython.a that can be used to build an application program
* an executable, `mpex.out`, in the ports/ti/boards/YOUR_BOARD directory

### Example Application

* flash mpex.out to board
* use REPL via UART terminal

## Status

* SD Card support with FAT filesystem
  * will execute /main.py if present when booted
  * Python module search path includes /modules directory
* UART REPL over XDS110 USB connection
* socket - socket API for both WiFi and NDK stacks
* network - control API for both WiFi and NDK stacks
* Hardware Modules - see machine modules

## machine Modules

* Follows ["machine" interface](https://docs.micropython.org/en/latest/pyboard/library/machine.html)
* Pin (GPIO) - read/write pins
* I2C - uses I2C driver configured in board.c
  * fixed to max 3 currently
* SPI - uses SPI driver configured in board.c
  * fixed to max 3 currently
* UART
* PWM - uses PWM driver configured in board.c

* deinit(): does nothing since TI-RTOS drivers have power management
  * UART closes underlying driver - only one instance of each UART id

### SD

A custom machine_sd.c is provided since there are HW SD drivers in TI-RTOS. This is used for both SPI and HW implementations.

## Tests

See the `ports/ti/tests` directory for basic tests for each module/class. These have been primarily tested on CC3220SF and MSP_EXP432E401Y and may require tweaking for other boards. These could be adapted to use the board name (as returned by `os.uname()` to set board-specific values.

## Questions

* init()/deinit() - lifecycle model
* release a resource/object - how/when call I2C_close()?
* multiple instances of same underlying resource are shared - appears to
be the MP model. Multiple creates (opens) of the same id/index as in

``` python
from machine import I2C

i2c1 = I2C(0, baudrate=100000)
print(i2c1)
<machine_i2c.20000448> id=1 baudrate=100000
i2c2 = I2C(0, baudrate=400000)
print(i2c1)
<machine_i2c.20000448> id=1 baudrate=400000
```

results in the last one setting the operating values for all the others.

* static tables are setup for N instances of each driver - these are currently hard-coded to the magic number "3" (see NUM_SPI and NUM_I2C)
