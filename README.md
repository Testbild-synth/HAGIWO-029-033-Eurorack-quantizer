# HAGIWO 029/033 Eurorack Quantizer Module
<img src="images/in_rack.jpg" width="30%" height="30%">
Through hole PCB version of the [HAGIWO 033 dual quantizer](https://www.youtube.com/watch?v=6FJpljEYZq4) and [029 SH101 sequencer](https://www.youtube.com/watch?v=--qb_QYZrTk) Eurorack Module. The modules uses a seeduino xiao and a mcp4725 dac.

The dual quantizer quantizes to notes selectable on the screen and can automatically trigger an envelope on note changes.
The SH101 sequencer allows recording and playback of cv sequences.

## CURRENT STATE: HARDWARE VERIFIED WORKING, DOUBLE QUANTIZER FIRMWARE WORKING, SH101 SEQ NOT TESTED


## Hardware and PCB
<img src="images/front_1.jpg" width="30%" height="30%"><img src="images/front_2.jpg" width="30%" height="30%">

You can find the schematic and BOM in the root folder.
For the PCBs, the module has one main circuit PCB, one control circuit PCB and one panel PCB. 
You can order them on any common PCB manufacturing service, I used [JLCPCB](https://jlcpcb.com/).
I made the circuits pcbs under 100mm to get the discount price.
Standard settings should be fine, but as there is exposed copper on the panel you should go with a lead free surface finish (ENIG/Leadfree HASL).
If the panel size is not correctly detected by JLC (happens on some of my exports) manually put 30x128.5 mm.


when ordering the display module, make sure to choose an 0.96 I2C oled module that has the pinout specified as GND-VCC-SCL-SDA as opposed to VCC-GND-SCL-SDA (both exist and the latter will fuck it up).    

Also make sure you order a Seediuno XIAO (with a SAMD21/Cortex M0 chip) as opposed to the XIAO esp32c3 or the XIAO rp2040, those are different chips.

<img src="images/display.jpg" width="20%" height="20%">

## Assembly

When assembling, you can either use a header for the screen or solder it directly, as it is a litte too tall.
The 7805 voltage regulator is optional, if you do not want to use it, simply solder the SEL header on the back of the main pcb to BOARD instead of REG (meaning you bridge the connection to choose your 5v voltage source to either be 12 regulated to 5v, or a 5V connection of your rack power if you have it).

## Tuning
There are two things  in the circuit that need to be tuned for: The input resistor divider going from 5V to 3.3V, and the output opamp gain to go from 3.3 back to 5V.
I will provide a script to help with this and a detailed description in the future, but for now the short version is: Input resistors are compensated for in code with the ADC_calb parameters. Output gain is adjusted using the trimmers on the main pcb.


## Arduino Code

### Double Quantizer
HAGIWO did great work, but I decided to make some changes to the Arduino code because I had ADC issues on my seeduino Xiao.

The main changes are:
 - HAGIWO had a check that cv is only updated then there's a big enough change from the last value. I took this out since I want to be able to process slow cv changes (i.e. from LFOs), also. 
 - I followed suggestions from [this very nice blog about adc accuracy on samd21](https://blog.thea.codes/getting-the-most-out-of-the-samd21-adc/). With lower input impedance and the changes from this blog, the readings got a lot more accurate on mine. Downside is more latency (in the single ms range) but frankly im willing to take that for more stability/less noise.
 - Also, to make use of this, the note calculation is now done with 12 bit instead of downsampling the adc values to 10 bit.
 - Comment out the specified lines in the code if you dont want the slower adc.

## TODO:
- Test and verify SH101 firmware
- maybe add display indication for played notes

