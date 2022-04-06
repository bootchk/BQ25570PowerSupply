BQ25570PowerSupply

A breakout board for Texas Instrument's BQ25570 energy harvester, with an additional LDO regulated 3.6V output.


### Overview

The board is a power supply.
It harvests energy, typically from solar cells.
It typically stores energy on a supercapacitor.

The board is designed for nano power applications,
where loads are powered off.
It includes load switches for two power rails.


### Rails

The power rails include:


TODO format this
    Unregulated storage rail, 0 to 5.5V.  VOUT
    Switched storage rail, 0 to 5.5V      VOUT_SWITCHED
    Regulated MCU power, 1.9 to 3.6V.     VREG
    Switched MCU power, 0 to 3.6V,        VREG_SWITCHED


### Status

Shared at OSHPark.  Minimally tested for design flaws, but not for other performance specs such as leakage.

Needs revision: 

The separate voltage monitor design is flawed.  The Vmon logic output ranges to 5.5V.  It needs a level converter to the same logic levels as the MCU.  In the interim, just don't populate the Vmon.

The LDO might be a SOT-23-6 package, for easier fabrication or choice of regulated voltage.

Obsolete the Good pin output.  Rename the pins.

Test pads.


### Use case

Intended for a nano power application.

The regulated output is for a low power MCU which sleeps mostly.  E.G. a MSP430 that draws 1mA active and say 50nA when sleeping.

The switched regulated output is for low power peripherals, to be powered off when not in use.  E.G. sensors.

The storage rail output pin is normally not used.

The switched storage output is for higher power, higher voltage 5V peripherals, to be powered off when not in use.  E.G. LED's, radios, or motors.


### References

The design follows this:  "Fast-charging a supercapacitor from energy harvesters" 
https://www.edn.com/fast-charging-a-supercapacitor-from-energy-harvesters/
In short, the supercapacitor is on the VOUT pin of the BQ, instead of the VBAT pin.
This makes coldstart relatively short in duration when charging a supercapacitor.


## Power States and Sequencing

### Coldstart state

Storage is discharged.
The BQ operates in coldstart mode.
It biases the solar cell at 0.6V (not the maximum power point MPT.)
Charging is relatively inefficient.
Voltage on supercapacitor storage remains near zero.

### Warmstart state

Voltage internal to the BQ reaches the VCHGEN voltage of the BQ (1.8V.)
Charging is relatively efficient, biasing the solar cell to its MPT.
The buck converter of the BQ pulses energy to the storage supercap.
Voltage on storage supercap slowly rises from zero.
See more details at the BQ25570 datasheet.

VREG and VREG_SWITCHED are off, at 0V.

Ordinarily, in this state you do not use power from the VOUT pin.

### Booted state

Voltage on the storage reaches the programmed Vbat_ok voltage (2.1V.)
The VBAT_OK pin of the BQ asserts high and enables the LDO.
The LDO begins operation in the linear mode, yielding  more than 2.0V (after subtracting Vldo of the regulator, less than 0.1 V.) 

Any MCU on the VREG pin powers on (boots.)
It should almost immediately go back to sleep until storage and voltage increase.
There is hysteresis.
The BQ will assert its VBAT_OK pin
at a slightly higher voltage (Vbat_ok_hyst) than the voltage (Vbat_ok) at which it deasserts VBAT_OK.
So there is 0.1V on the capacitor to carry the MCU through booting.

Voltage on storage, VOut, and VReg will continue to rise.

The MCU can read its own Vcc voltage to know when the system is in this state.
The MCU can read its own Vcc voltage to know when it is nearing brownout.

The MCU boots at rising voltage 1.9V but will not brownout reset until falling voltage 1.7V.

### Regulation state

Voltage on the storage reaches the set point of the LDO (3.6V.)
The LDO regulates (limits) voltage on VReg to 3.6V.
The MCU Vmax is 3.6V.

Voltage on storage continues to rise.
Additional voltage and energy on storage is "reserve."

The MCU can read its own Vcc to know when the system is in this state
(when Vcc is near 3.6V.)
The MCU can control energy use according to its own Vcc.
For example, the MCU can switch on high-power peripherals
only in this state.
Or it can use high-power peripherals down to a Vcc voltage between 2 V and 3.6 V.


### Full

Voltage on storage is the max that the BQ will provide, 5.5V.
The supercap must be rated for this voltage.

The MCU cannot know how much excess voltage is on storage.
The MCU can only know that if its Vcc is near 3.6V, there might be excess voltage on storage.



## Pinout

    Sol1           Voltage in, positive terminal of solar cell
    GND            Negative terminal of solar cell

    Supercap1      Positive terminal of storage supercapacitor
    GND            Negative terminal of supercap

    VOut           Unregulated storage rail, 0 to 5.5V.  Same net as Supercap1.
    VOutSw         Switched storage rail, 0 to 5.5V
    OutSw          Logic input, high to switch VOutSw.
     
    VReg           Regulated MCU power, 2.0 to 3.6V.     
    VRegSw         Switched MCU power, 0 to 3.6V,   
    RegSw          Logic input, high to switch VRegSw     

    Good           Logic output, high when VReg is > 2.0V.  Same net as BQ VBAT_OK.


## Programming 

### Programming the BQ 25570

Many voltages cited above are due to the programming of the BQ25570.

The programming can be changed by choosing resistors in the programming nets.

These are the voltages for the BQ programming resistors in the design (in the bill of materials.)

    V_OV          The overflow voltage of the BQ boost charger     5.5V (the max the BQ will output)

    VOUT          The regulated voltage of the BQ buck converter   5.5V

    V_BAT_OK      The rising voltage at which VBatOk asserts       2.1V
    V_BAT_OK_HYST The falling voltage at which VBatOk deasserts    2.0V

(The undervoltage of the BQ is hardwired in the BQ.)

Again note that the BQ buck converter/regulator is not used in the usual fashion,
but to charge the supercapacitor.

### Programming the LDO

The regulated voltage of the LDO is determined by the part number of the LDO.
(Set at the factory.)

## Programming for the MSP430

The BQ is programmed with the MSP430 in mind.
Since the MCU is on the LDO output rail (VReg), and the LDO has a voltage drop Vdo of about 0.1V at 1mA,
the voltages programmed for the BQ V_BAT_OK are 0.1V higher than the voltages the MCU sees.

### Boot voltage

The BQ asserts Vbat_ok signal at 2.1V, when MCU sees 2.0.
This is greater than the the MSP430 Vsys+ (1.99V max) , the voltage at which the MSP430 is guaranteed to boot (come out of reset.)

### Reset voltage

Furthermore, this board is designed to allow the MSP430 to enter its very lowest power state.
That state is LPM4.5 with SVS shut down ( a substate of LPM4.5,) drawing only 16nA.

The SVS is the system voltage monitor that can help ensure proper reset.
In the mode with SVS disabled, the MSP430 draws less current.
But disabling SVS requires careful consideration of brownout reset requirements.

The requirements for a safe reset when SVS is disabled:
(from the MSP430 datasheet) 

    Vcc less than Vbor (0.1V)
    Vcc greater than Vbor for time tBOR_safe (10 mS)

If these requirements are not met, the MSP is not guaranteed to come out of reset after a voltage drop.

In summary, in this design, the BQ25570 signal Vbat_ok cycles (switches on and off)
power (the LDO) to the MSP430 MCU,
so as to meet the above requirements.

When Vbat_ok goes low at level VBAT_OK (2.0V) , the LDO is disabled.
Voltage and current from the LDO drops to zero.
As it drops below MSP Vsys- (1.7V min),
the MSP enters the reset state, drawing 50uA.

Typical filter capacitors on the MCU are 10uF.
At 50uA current, the filter caps discharge in less than a millisecond.
Thus voltage on the MSP Vcc pin quickly drops below MSP Vbor of 0.1V.

In this condition, the other requirement is to stay in this condition for time tBOR_safe.
Since voltage on the supercap is at level VBAT_OK (2.0V)
the voltage must rise to level VBAT_OK_HYST (2.1V)
before the BQ raises Vbat_ok again and enables the LDO.

As an example, assuming a 1F supercap and 3mW from the solar cell (say a Powerfilm LR3-200-37 at 1000 lux)
it takes 0.2 seconds to charge the supercap from 2.0 to 2.1V.
Said time is greater than the requirement.
Other choices ( using a smaller supercap and larger solar cell)
must consider the same requirements.


## Max current ratings

VReg          LDO             (TI part TSP7A02)            200 mA

VOutSwitched  LoadswitchOut   (TI part number TPS27081A)     3 A
    
VRegSwitched  LoadswitchReg   (TI part number TPS22860)    200 mA



## Recommmended operating conditions and LDO dropout

The recommended load on VREG is only a few mA.
The recommended load is very much less than the rated load of the parts.
An MSP430 MCU at 1 Mhz uses only 1mA.

The voltage seen on the VREG net depends on the dropout voltage of the LDO (Vdo)
which depends on load current.

For conditions:

    - temperature -40 to +85 degrees C
    - current 1 mA
    - voltage out 3.6V

the LDO datasheet shows typical Vdo of 0.08V.

The programming of the BQ Vbat_ok signal 
and the choice of MCU depends on such a small Vdo of say 0.1V.

If the MCU on VREG and peripherals on VREG_SW use much more power than say 10's of mA,
you should consider the effect on LDO dropout,
and not switch the peripherals unless Vreg is higher than say 2.3V as determined by the MCU.
Pulling a higher load might increase the Vdo and lower the voltage seen by the MCU,
possibly enough to brown it out.


## Pullups/Pulldowns on signals

The board has a pullup resistor RVoutPull on LoadswitchOut (TI part number TPS27081A).
Note this is per the example for standby power in the LS datasheet.
The pullup is NOT on the enable input, but to between the mosfets on the load switch.
This pullup keeps the loadswitch off while voltage is rising.

The board has no pullup or pulldown on the enable of LoadswitchReg   (TI part number TPS22860).
The loadswitch is on the same rail VReg as the MCU, switched by the MCU.
The MCU should early on set the enable low, and set it high only momentarily.

The LDO has a "Smart enable" feature to ensure it is shutdown during power up.


## Quiescent current of the power supply

Quiescent current is the current out of storage when any attached load is not drawing current.
Quiescent current subtracts from any charging current (say from the solar cell.)

The storage leaks when being charged and also self discharges when not being charged.
Typically this current dominates.
Typically this current is on the order of 1 uA (microamps).
This current depends on the capacitance in F among other things like the supercap technology.
I exclude this current from quiescent current.
But the solar cell current must provide this current to keep storage charged.

### Before VReg in regulation

The MCU is not booted so doesn't drive the loadswitch enable pins.

    LoadswitchOut is shutdown(standby)     50 nA
    LoadswitchReg is not powered            0 nA
    LDO is shutdown                         3 nA

    total                                  50 nA

LoadswitchOut has a feature "Smart Enable" to ensure it is not enabled on voltage rising during powerup.

### After VReg in regulation

The MCU is booted but sleeping mostly, and holding loadswitches off.

    LoadswitchOut is standby               50 nA
    LoadswitchReg is powered but shutdown  12 nA
    LDO is regulating                      25 nA
    MCU is booted but in low power mode    50 nA

    total                                 137 nA







