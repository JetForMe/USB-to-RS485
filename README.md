# High Speed USB to EIA-485 Adapter

I got fed up trying to find a USB-to-EIA485 adapter that could do 4 Mbps or better. Most
of the available FTDI and CP2102N ICs top out at 3 Mbps, and most of the available
EIA-485 driver ICs are slew rate-limited to well below 4 Mbps.

So I’ve created this adapter. Some of its features include:

* USB-C
* 12 Mbps maximum speed
* 5 V, 3 A power output
* 3.3 V, 600 mA (including the adapter itself) power output
* Configurable USB-C PD output, up to 20 V @ 100 W (hopefully)
* FTDI-compatible 3.3 V 6-pin UART header
* 3.3 V or 5 V selectable, ±70V fault-protected EIA-485 driver (PROFIBUS compatible)
* Full- or half-duplex operation
* Optional bus termination resistors
* EIA-485A and EIA-422B compatible
* JTAG programmer/debugger
* Interface to many other serial busses (I2C, SPI, etc.)
* All FT232HP GPIO pins brought out on headers

[Schematic](/hardware/Schematic.pdf)

## macOS Notes

To set non-standard UART speeds on macOS, you must use `ioctl()` after calling `tcsetattr()`:

In C/C++:

```cpp
#include <sys/ioctl.h>
#include <termios.h>

#define IOSSIOSPEED _IOW('T', 2, speed_t)		//	Defined in <IOKit/serial/ioss.h>
speed_t speed = 4000000;
if (ioctl(fd, IOSSIOSPEED, &speed) != 0)
{
    perror(__FILE__ ":" STRINGIZE(__LINE__) ": ioctl set speed");
}
```

From [IOSerialTestLib.c](https://opensource.apple.com/source/IOSerialFamily/IOSerialFamily-74/tests/IOSerialTestLib.c):

```c
int _testIOSSIOSPEEDIoctl(int fd, speed_t speed)
{
    struct termios options;
    
    // The IOSSIOSPEED ioctl can be used to set arbitrary baud rates other than
    // those specified by POSIX. The driver for the underlying serial hardware
    // ultimately determines which baud rates can be used. This ioctl sets both
    // the input and output speed.
    
    if (ioctl(fd, IOSSIOSPEED, &speed) == -1) {
        printf("[WARN] ioctl(..., IOSSIOSPEED, %lu).\n", speed);
        goto fail;
    }
    
    // Check that speed is properly modified
    if (tcgetattr(fd, &options) == -1) {
        printf("[WARN] _modifyAttributes: tcgetattr failed\n");
        goto fail;
    }
    
    if (cfgetispeed(&options) != speed ||
        cfgetospeed(&options) != speed) {
        printf("[WARN] _modifyAttributes: cfsetspeed failed, %lu, %lu.\n",
               speed,
               cfgetispeed(&options));
        goto fail;
    }
    
    return 0;
fail:
    return -1;
}
```


Swift version to come! (_IOW defined in <sys/ioccom.h>)


