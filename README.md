# Pool Control
Project using ESPHome on a Waveshare 8CH Relay to Control a Hayward TriStar Filter Pump, a Pentair Spa Jets pump, and 2 Pentair Pool Lights for both local control and integration with Home Assistant.

## Goals
Control exsiting equipment both locally at the junction box and remotely via Home Assistant.
- Full local control in the event that the network or HA are unavailable.
- Use WiFi with reliable signal (external antenna)
- Take advantage of the "Remote Relay Control" option of the TriStar pump to control speed and on/off.
- Operate the 220v relay to control the Pentair Pump.
- Operate the Pool Lights
  - Locally and remotely turn on/off the pool lights
  - Locally and remotely interrupt each of the LED lights to set their colors or "color shows".

## Existing Equipment
- \(1\) Hayward [TriStar Variable Speed Pump](https://hayward.com/tristar-vs-900-sp32900vsp.html) (220VAC)
- \(1\) Pentair Superflow 2hp Pump (220VAC)
- \(2\) Pentair [IntelliBrite Color LED Pool Lights](https://www.pentair.com/en-us/pool-spa/products/lighting/intellibrite-5g-color-led-pool-lights.html) (24VAC)

## Materials
- \(1\) - Waveshare 8CH Relay Board \([ESP32-S3-ETH-8DI-8RO-C](https://www.waveshare.com/wiki/ESP32-S3-ETH-8DI-8RO-C)\).
- \(7\) - 12mm Metal Waterproof Switch Momentary Push MINI Button Switch \(Similar item: [Momentary Switch](https://www.ebay.com/itm/265398183120)\).
- \(1\) - 30A/220V Power Relay with 110VAC Coil \(Similar item: [Baomain High Power Relay](https://www.ebay.com/itm/286505847153)\).
- \(1\) - Qilipsu Junction Box with Mounting Plate \([QL-251510AG](https://www.qilipsu.com/product-204.html)\).  Note: This particular junction box is plastic, make sure you are comfortable with using a plastic enclosure as other options in metal or fiberglass exist.
- Miscelaneous 12AWG cable, Sealtite conduit, 24AWG solid core wire, etc.

## Version Info
The final product was developed with:
- ESPHome 2025.6.3
- Home Assistant (HA) 2025.7.1.

## Notes
### The YAML Code
This is litteraly my first significan dive into ESPHome and YAML.  The MUST be many ways to improve the code and remove bloat \(i.e. passing parameters to scripts... * sigh... *\).  Any helpful feedback for improving the code is appreciated and welcomed; any rude feedback will simply be deleted.

### Hayward TriStar Variable Speed Pump
For do-it-yourself'ers, Hayward does not seem willing to share information about their implementation of MODBUS.  Some people have tried to sniff MODBUS traffic for these pumps \(i.e. [Desert Home](http://www.desert-home.com/2014/07/controlling-hayward-ecostar-pump.html)\) and have figured out a bunch, but that's not a project I'm willing to take on at the moment. 

Consequently, I'm leveraging the Remote Relay Control feature which, simply put, uses 3 binary inputs to create 8 possible selections for a speed \(INP1, INP2, INP3\).  The 8 speeds come from programming each of the 8 timers in the pump.  INP4 is used to turn the pump off/on... but note that it is reversed so you'll want to make sure to use the normally closed \(NC\) side of the relay for that one so the pump is off when the Waveshare device is without power.

Since I want to run most often at the highest speed configured \(I really only run my filter pump for 1 hour a day\) and I would want that to happen when the relays for INP1, 2, and 3 are resting rather then holding them on for an hour each day, I reversed the order of the speeds with Timer-1 being the fastest speed and Timer-8 being the slowest.  

After setting each of the Timers for their speeds, I started the pump, allowed it to settle and read the wattage directly from the pump.  It's hard coded and one of the GREAT reasons that using MODBUS would have been better and I probably could have read that wattage on the fly rather than depend on something hard coded.  Noththeless, this allows me to create a measurement sensor that I can the use by and energy sensor to tally my usage in HA.

| Input  |  T1  |  T2  |  T3  |  T4  |  T5  |  T6  |  T7  |  T8  |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| INP1   |  OFF |  ON  |  OFF |  ON  |  OFF |  ON  |  OFF |  ON  |
| INP2   |  OFF |  OFF |  ON  |  ON  |  OFF |  OFF |  ON  |  ON  |
| INP3   |  OFF |  OFF |  OFF |  OFF |  ON  |  ON  |  ON  |  ON  |
| Programmed Values |
| RPM    | 3400 | 3200 | 3000 | 2800 | 2600 | 2400 | 2200 | 2000 |
| Watts  | 1250 | 1050 | 850  | 680  | 540  | 430  | 335  | 260  |

Side-note, it makse no differenct if the timer is enabled or disabled when the unit is in Remote Relay Mode... either way, the timers will not run on their own and you will need to handle that via Home Assistant.  Also note, once you are in Remote Relay Control mode, you'll need to turn on the pump to be able to change the timers.

### Pentair Intellibright 5G Color LED Pool Lights
Honestly, I have been very, very happy with these lights for the last 10 years or so.  They have a number of modes and colors that you can set by first turning the light on, then cycling the power off then on for any of 1-14 times.  I have always wanted to have automation at the pool \(i.e. not dependent on a smarthome product\) that could handle the setting of each of those modes, then trigger that automation from HA.  Below are the modes and number of times to cycle each one... as well as a couple name changes I took the liberty of making.

| Cycle Count | Original Name | Name in Code | Description |
| :---------: | :-----------: | :----------: | ----------- |
| 1 | SAm Mode | SAm | Cycles through white, magenta, blue and green colors (emulates the Pentair SAm® color changing light). |
| 2 | Party Mode | Party | Rapid color changing building energy and excitement. |
| 3 | Romance Mode | Romance |  Slow color transitions creating a mesmerizing and calming effect.|
| 4 | Caribbean Mode | Caribbean | Transitions between a variety of blues and greens. |
| 5 | American Mode | Red White & Blue | Patriotic red, white and blue transition. |
| 6 | California Sunset Mode | California Sunset | Dramatic transitions of orange, red and magenta tones. |
| 7 | Royal Mode | Royal | Richer, deeper color tones. |
| 8 | Blue | BLue | Fixed color. |
| 9 | Green | Green | Fixed color. |
| 10 | Red | Red | Fixed color. |
| 11 | White | White | Fixed color. |
| 12 | Magenta | Magenta | Fixed color. |
| 13 | Hold | Capture |  Save the current color effect during a color light show. |
| 14 | Recall | Recall Last | Activate the last saved color effect. |

Still To-Do on this might be to add a 15th cycle of the button that would hold the relay on (contact open) to allow me to turn off one or the other of the lights.  Some thought will need to go into identifying when a) power is on to the lights, but b) the cycle inturrupt is open when setting a new choice for the mode.  Granted I would do it easy for HA control only, but adding local control without adding more buttons is the reason for a 15th power cycle.

### The Qilipsu Junciton Box
Plastic is not the best choice for this box with the high-voltage included, but it's something I can look to replace donw the road.  I would even love to find a way to better separate the high and low-voltage a bit better.

### Final Thoughts
I am super happy with the results of this project.  The unit is extremely responsive, fully local not just in terms on leveraging Home Assistant, but also how ESPHome has let me put all of the functionality (except scheduling) on the device itself so I can run it standalone.  The Waveshare 8CH device does have an RTC, so I suppose I could work towards building in a scheduling feature, but again... that's just out of scope at this point.

## Wiring Diagram
![Wiring Diagram](https://github.com/jeparkspr/pool-control/blob/main/wiring-diagram.jpg)

## Finished Product
![Finished Product](https://github.com/jeparkspr/pool-control/blob/main/finished-product.jpg)
