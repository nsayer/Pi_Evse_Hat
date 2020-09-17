# Pi_Evse_Hat
Software for https://hackaday.io/project/167595-raspberry-pi-evse-hat

The Pi EVSE hat is a board with a number of separate subsystems combined to add up to everything required for an electric vehicle charging station (EVSE).

The code here is in Python, which meshes properly with the Raspberry Pi GPIO Python library. In addition, we require the Python SPI library (and for the SPI bus to be enabled) to talk to the ADC chip, as well as hardware PWM support. Alas, there is no Python library for that. We need to manually open files in /sys/class (or perhaps write our own library).

To review, the GPIO pins to talk to the hat are (these are GPIO numbers, not pin numbers):
* 4 - The power watchdog pin. Whenever the relay is powered on, you must toggle this pin at 1-10 kHz.
* 17 - The power pin. Set this high to turn the vehicle power on. Whenever it is on, you must toggle the power watchdog pin or else a (synthetic) GFI event will occur.
* 18 - the pilot pin. Set this high for +12, low for -12. See below for using this pin.
* 22 - The GFI status pin. This is an input. When high, the GFI has been set and turning the power on is disallowed.
* 23 - The relay test pin. This is an input. It gives the status of the HV relay / GCM test. Within 100 ms of turning the power on, this pin should go high and stay high. Within 100 ms of turning the power off, this pin should go low and stay that way.
* 24 - The GFI test pin. This is an output pin. A wire connected to the GFI test pin should take two loops through the GFI CT and then connect to ground. Toggle this pin at 60 Hz to simulate a ground fault as part of the GFI test procedure.
* 27 - The GFI reset pin. Pulse this pin high to clear the GFI. You cannot do this while the vehicle power is turned on.

In addition to the above, the MCP3004 ADC is connected to the SPI system, on device zero (/dev/spidev0.0). The first three channels are connected to:

* 0 - the pilot feedback. You should sample this a few hundred times, taking the high and low values found. The high value can be used to detect vehicle state changes. The low value should remain near -12v (when the pilot is oscillating) or else it's a "missing diode" test failure.
* 1 - the current sense transformer. You should sample this pin to obtain samples across two zero crossings (512 is zero), perform an RMS calculation, and scale the result to determine how much current the vehicle is drawing.
* 2 - the AC voltage sense. You should sample this looking for the peak. This will allow you to determine the AC voltage and with the current sense, determine how much power the vehicle is drawing.

You can turn SPI on with raspiconfig, but to enable hardware PWM on pin 18, you must add this to /boot/config.txt:

`dtoverlay=pwm,pin=18,func=2`

