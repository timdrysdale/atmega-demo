# DESIGN DESCRIPTION TEMPLATE

## Background

This board is to demonstrate self-assembled Atmega328P circuit.

## Specification

Arduino with USB programming, current sensor, and temp sensor.

Possibly reverse polarity protection, and/or level shifter and external connections headers.

### Design tradeoffs

Note any issues which could be handled in more than one way and justify the approach taken?

### Design for assembly

How will you put it together e.g. if using cable connections, how will these be manufactured?

### Design for reliability

Things go wrong with assembly and usage, even with the best of staff working on the job. So how will your design prevent itself from being damaged by accidental or even malicious misuse, e.g. reversed power supplies, or wrong power supply, incorrect peripheral connections etc (especially where terminal blocks are used, because polarity cannot be enforced very easily).

### Design for maintenance 

What parts are likely to fail, and how will they be replaced?


## Part  details

List here any helpful details which can assist in the review process, such

### Atmega 328P-AU

JLCPCB Part# C14877
Package TQFP-32_7.0x7.0x0.8P
Price ca. $6

KiCAD footprint: Package_QFP:TQFP-32_7x7mm_P0.8mm

Added LCSC code to part in schematic

TXD PD1
RXD PD0

#### Clock

Since want to use USB, need a crystal (else [reported](https://electronics.stackexchange.com/questions/27763/using-the-atmega328-with-the-internal-oscillator) 10% variation in the clock frequency, i.e. outside acceptable range for USB clock speeds.

Internal oscillator calibration procedure is [here](http://projects.gctronic.com/elisa3/AVR053-Calibration-RC-oscillator.pdf) for reference - don't use that for now though.

There are some suggested oscillator frequencies
3.6864MHz (0% error for 230.4k Baud and under)
4.0000MHz (0.2 - 8.5% errors)
7.3728MHz (0% error for 230.4kBaud and under)

Some issues with clocks >8 MHz depending on VCC - to avoid issues, try 4 MHz clock for now?
Arduino uno uses Atmega 328p with 16MHz crystal....


### Oscillator

p146 Table 19.1 has formulae for calculating baud rates.

Async mode, normal rate, BAUD = f_OSC / (16* (UBBRn + 1))

UBBRn has a range from 0 to 4095.

p162 Table 19.9 has some common crystal frequencies

f_OSC= 14.7456 MHz
UBBRn = 15
BAUD = 57k6
Error = 0.0%

Check...

14745600 / (16 * (15+1)) = 57600

Now try a 12MHz crystal ...

guess at UBBRn = 12

12000000/(16*(12+1)) = 57692


Error as per section 19.11

(57692/57600 - 1) * 100%  = 0.1%

0.5% is acceptable, and more than that just affects noise resistance ... so let's try it...

So we can use the same crystal as for the CH340....


#### 3.4 MHz option
Yangxing Tech X49SM36864MSD2SC-1

JLCPCB Part # C2238
Package HC-49S
Description -20℃~70℃ 49SMD ±20ppm 20pF 150Ω 3686400Hz HC-49S/SMD Crystals ROHS
SMT Assembly
Price 6c / min 5
Extended part

#### 14.7MHz option

also good is 14.7456MHz - no errors for most baud rates.

but put in 18pF-22pF caps as suggested [here](https://www.arduino.cc/en/Tutorial/BuiltInExamples/ArduinoToBreadboard)

14.7456MHz oscillator of same package is C240976

### Oscillator capacitors

since 3 - 8 MHz oscillator, Table 8-3 says want 12 - 22pF

![xtal](./img/xtal.png)

Use 0805 size as lots in stock.

Manufacturer: 0805CG120J500NT

12pF is C1792 Basic part 

Footprint: Capacitor_SMD:C_0805_2012Metric

But upgrade to 18pF for higher frequency crystal, LCSC part: C237185


Upgrade again ... to 22pF because that is what the CH340G needs...
And go to 0603 size seeing as we can.
Manufacturer
Samsung Electro-MechanicsCopy
MFR.Part #
CL10C220JB8NNNCCopy
JLCPCB Part #
C1653Copy
Package
0603Copy
Description
C0G ±5% 50V 22pF 0603 Multilayer Ceramic Capacitors MLCC - SMD/SMT ROHSCopy
Datasheet
Download
Source
JLCPCB
Assembly Type
SMT Assembly
EasyEDA Libraries
PCB Footprint or Symbol


### Reset switch
Manufacturer C&K
MFR.Part # PTS810SJM250SMTRLFS
JLCPCB Part # C116501
Package KEY-SMD
Description Top Actuated 50mA @ 16VDC Round Button SPST SMD Tactile Switches ROHS
Source LCSC
Assembly Type SMT Assembly

Footprint dimensions checked manually
Button_Switch_SMD:SW_SPST_PTS810

### pull up (now changed to 10K)

Revised version: 10K, 0603

Manufacturer
UNI-ROYAL(Uniroyal Elec)Copy
MFR.Part #
0603WAF1002T5ECopy
JLCPCB Part #
C25804Copy
Package
0603Copy
Description
±1% ±100ppm/℃ 10kΩ 0.1W 0603 Chip Resistor - Surface Mount ROHS



First go: 5K
C78980

250mW

0805



### Analog filter

![analog](./img/analog.png)

100nF cap C49678 0805 basic part
footprint:Capacitor_SMD:C_0805_2012Metric

10uH inductor C1046 basic part
(confirmed value by checking part code in table in datasheet SDFL2012S100KTF)
footprint: Inductor_SMD:L_0805_2012Metric


### ICSP header

3x2 2.54mm pitch, programmer such as [this](https://www.pololu.com/product/3172)

Pin 1 MISO
Pin 2 VCC
Pin 3 SCLK
Pin 4 MOSI
Pin 5 RST(negative logic)
Pin 6 GND

Connections: as per datasheet (no connection to XTAL1 says Arduino Uno reference design)
p 254
Table 27-15. Pin Mapping Serial Programming

Symbol Pins I/O Description
MOSI PB3 I Serial data in
MISO PB4 O Serial data out
SCK PB5 I Serial clock

#### CH340

G version in stock

needs crystal of 12MHZ

can we use that for the arduino too, and save a reel fee?


C97242

#### 12 MHz crystal for the USB UART

Manufacturer
TXC CorpCopy
MFR.Part #
7V12000018Copy
JLCPCB Part #
C97242Copy
Package
SMD-3225_4PCopy
Description
12MHz ±30ppm 20pF 100Ω SMD-3225_4P SMD Crystal Resonators RoHSCopy
Datasheet
Download
Source
JLCPCB
Assembly Type
SMT Assembly

This needs either 22pF caps (most examples) or 33pF if powered USB as well. No clear statement as to why has been found in the datasheet yet - but we do it the way it is shown.

Manufacturer
Samsung Electro-MechanicsCopy
MFR.Part #
CL10C330JB8NNNCCopy
JLCPCB Part #
C1663Copy
Package
0603Copy
Description
C0G 33pF ±5% 50V 0603 Multilayer Ceramic Capacitors MLCC - SMD/SMT ROHS

### USB Micro B receptable

The ID pin is mode detect - leave NC for normal peripheral mode, [see here](https://pinouts.ru/PortableDevices/micro_usb_pinout.shtml)

Amphenol ICC
10118194-0001LF
C132563
Connector_USB:USB_Micro-B_Amphenol_1011

### CH340 power caps

0.1uF is 100nF, can get that in 0603

Manufacturer
YAGEOCopy
MFR.Part #
CC0603KRX7R9BB104Copy
JLCPCB Part #
C14663Copy
Package
0603Copy
Description
X7R 100nF ±10% 50V 0603 Multilayer Ceramic Capacitors MLCC - SMD/SMT ROHS

### USB data line protection ....

do we need this with the CH340?? It is ESD, may as well have it.

Manufacturer
NexperiaCopy
MFR.Part #
PRTR5V0U2X,215Copy
JLCPCB Part #
C12333Copy
Package
SOT-143Copy
Description
Unidirectional - - 6V 5.5V SOT-143 ESD Protection Devices ROHSCopy
Datasheet
Download
Source
LCSC

1M resistor


Manufacturer
UNI-ROYAL(Uniroyal Elec)Copy
MFR.Part #
0603WAF1004T5ECopy
JLCPCB Part #
C22935Copy
Package
0603Copy
Description
1MΩ ±1% ±100ppm/℃ 0.1W 0603 Chip Resistor - Surface Mount ROHSCopy
Datasheet
Download
Source
JLCPCB
Assembly Type
SMT Assembly

#### USB connection to Atmega
see o70 Table 13-9
PD1 TXD (USART output pin)
PD0 RXD (USART input pin)

#### Test point

KeystoneCopy
MFR.Part #
5017Copy
JLCPCB Part #
C238130Copy
Package
TEST-SMDCopy
Description
SMD Test Points/Test Rings ROHS

This has hte same footprint as the keystone 5015 micro miniature

TestPoint:TestPoint_Keystone_5015_Micro-Minature (3.43mm by 1.78mm)

#### ACS712



Manufacturer
Allegro MicroSystems, LLCCopy
MFR.Part #
ACS712ELCTR-20A-TCopy
JLCPCB Part #
C10681Copy
Package
SOIC-8_3.9x4.9x1.27PCopy
Description
SOIC-8_150mil Current Sensors ROHSCopy
Datasheet
Download
Source
JLCPCB


needs 100n decoupler (same as otheres)

an 1n filter 

Manufacturer
YAGEOCopy
MFR.Part #
CC0603KRX7R9BB102Copy
JLCPCB Part #
C100040Copy
Package
0603Copy
Description
X7R 1nF ±10% 50V 0603 Multilayer Ceramic Capacitors MLCC - SMD/SMT ROHSCopy
Datasheet
Download
Source
LCSC
Assembly Type
SMT Assembly

### LEDS

0603 

need 68Ohm ballast at 5V

C27592

Manufacturer
UNI-ROYAL(Uniroyal Elec)Copy
MFR.Part #
0603WAF680JT5ECopy
JLCPCB Part #
C27592Copy
Package
0603Copy
Description
±1% ±200ppm/℃ 68Ω 0.1W 0603 Chip Resistor - Surface Mount ROHS

#### Temp sensor

Manufacturer
Microchip TechCopy
MFR.Part #
MCP9701AT-E/TTCopy
JLCPCB Part #
C150830Copy
Package
SOT-23-3Copy
Description
-10°C ~ 125°C - Analog, Local 19.5mV/°C Analog Voltage SOT-23(SOT-23-3) Temperature Sensors ROHS


#### Sourcing

Where can you get it from?

#### Size, orientation

How big is it, and which way round does the footprint need to be on the PCB? How much clearance is needed around the part e.g. for putting in wires to terminal blocks

#### Supply considerations

What is the typical, and absolute supply voltage(s)?

#### Application considerations

What supporting components are needed? 

#### Electromagnetic compatibility considerations

Note any components which should be close together, far apart, and any track length considerations







