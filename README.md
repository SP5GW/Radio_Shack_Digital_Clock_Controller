# Radioshack digital clock controller

## Purpose of the project

This project has been started out ot practical need to control power supply to radio shack real time clocks so they are turned on/off in sync with PC monitor status.

To avoid clocks to be turned off when monitor briefly displays dark scene, delay timer is added, which provides option to ignore short periods of monitor getting either blank, being disconnected or put into sleep.

## Design challanges

Initially the focus was put on detecting the monitor state in software, which could then control hardware activating clocks power supply. However it quickly became apparent that it is not trivial (if possible at all) to realibly detect monitor status especially under Windows OS. There are several posts available on internet discrabing this challange, but all implementations examined proved to be not reliable at all. 

Next, the focus shifted to HDMI interface and possibity to build hardware status detector using signals available on HDMI bus. This idea also had to be dropped mainly due to High-bandwidth Digital Content Protection (HDCP) mechanism implemented on HDMI causing any tapering with HDMI signals very difficult.

Selected solution is based on AC current sensor, which detects increase in PC monitor power consumption, this increase can be associated with the monitor getting into active state. Solution is implemented in harware only.

## Operation description

Circuit diagram of presented controller can be seen below:

<p align="center">
<img src="./img/Schematics.png" width="1000" height="600"/>
</p> 

General method of attaching SCT-013-005 AC current sensor to the power line can be seen below (drawing curtuasy of Murky Robot):

<p align="center">
<img src="./img/sct-013-esquema-electrico.png" width="400" height="200"/>
</p> 

Due to the switching power supply of the PC monitor, the output signal Ua from the AC current sensor (SCT-013-005) consists of pulses with an amplitude of approximately ±100 mV and a frequency of 50 Hz. 

Before reaching the D1/C1 rectifier, the signal from the current sensor is amplified by the op-amp U2A (LM358) configured as a non-inverting amplifier, the gain of this stage is determined by resistors R3 and R4. With R3 = 38 kΩ and R4 = 1 kΩ, the gain is calculated as:

$G = 1 + (R3/R2) = 1 + (38k/1k) = 39$

The gain level of U2A is selected to produce output voltage well above forward diode D1 drop voltage of about 0.6V. In our case the amplitude of U2A output voltage Ub is:

$Ub = G * Ua = 39* 0.1V = 3.9V$

Signals Ua and Ub are shown on the picture below (Ua - blue curve, Ub - yellow curve):

<p align="center">
<img src="./img/initial_amplification_stage.png" width="400" height="200"/>
</p> 

Resistor R1 is included to prevent the output voltage of U2A from spiking to its maximum value when the current sensor is disconnected. When the sensor is connected, its internal resistor pulls the op-amp’s non-inverting input to ground. Without R1, disconnecting the sensor would leave the op-amp’s non-inverting input floating.

R1 should be significantly larger than the internal reference resistor of the current sensor to avoid affecting the sensor's output voltage.

DC voltage from the rectifier is compared to the reference voltage using op-amp U2B, configured as an inverting comparator.

The reference voltage is set to approximately 1V using the RV1 potentiometer. This value can be reduced if the output voltage amplitude from the current sensor is significantly lower than 100 mV.

When the PC monitor is active, the DC voltage at the rectifier output exceeds 3.6V. Since this is greater than the 1V reference voltage, the output of comparator U2B is driven close to 0V (TTL low state).

When the PC monitor is disconnected or in sleep mode, the DC voltage at the rectifier output is driven close to 0V. Since this is lower than the 1V reference voltage, the output of comparator U2B is driven close to Ucc (TTL high state).

Resistor R1 is added to allow capacitor C1 to discharge when the PC monitor transitions from an active state to a disconnected state, i.e. without R1, the rectifier output voltage would remain high even after the PC monitor enters sleep mode or is disconnected.

The signal from comparator U2B is inverted by the Schmitt trigger NAND gate U3A (4093) and then controls an RC circuit that introduces a delay adjustable from 0 to 6 seconds using potentiometer RV2. Diode D2 allows for immediate capacitor C2 discharge whenever U3A output goes to low state, resulting in no delay in clock power off as soon as monitor is disconnected or put into sleep.

Output voltage from RC circuit is inverted back by NAND gate U3B and then together with signal from push button SW2 steers NAND gates U3C/U3D, which control relay circuit activated by TTL low state.

Switch SW2 when on, permanently activates relay circuit leaving clocks powered on irrespective of PC monitor state. When SW2 is off clocks are only powered on when monitor is active.

Led 1 (green) is on, when the device is connected to AC power line.

Led 2 (blue) is on, when relay is activated (clock modules are powered on).


# References

