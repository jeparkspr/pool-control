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
### Hayward TriStar Variable Speed Pump
For do-it-yourself'ers, Hayward does not seem willing to share information about their implementation of MODBUS.  Some people have tried to sniff MODBUS traffic for these pumps \(i.e. [Desert Home](http://www.desert-home.com/2014/07/controlling-hayward-ecostar-pump.html)\) and have figured out a bunch, but that's not a project I'm willing to take on at the moment. Consequently, I'm leveraging the Remote Relay Control feature which, simply put, uses 3 binary inputs to create 8 possible selections for a speed \(INP1, INP2, INP3\).  The 8 speeds come from programming each of the 8 timers in the pump.  INP4 is used to turn the pump off/on... but note that it is reversed so you'll want to make sure to use the normally closed \(NC\) side of the relay for that one so the pump is off when the Waveshare device is without power.

Since I want to run most often at the highest speed configured \(I really only run my filter pump for 1 hour a day\) and I would want that to happen when the relays for INP1, 2, and 3 are resting rather then holding them on for an hour each day, I reversed the order of the speeds with Timer-1 being the fastest speed and Timer-8 being the slowest.  

Side-note, it makse no differenct if the timer is enabled or disabled when the unit is in Remote Relay Mode... either way, the timers will not run on their own and you will need to handle that via Home Assistant.  Also note, once you are in Remote Relay Control mode, you'll need to turn on the pump to be able to change the timers.

### Pentair Intellibright 5G Color LED Pool Lights
Honestly, I have been very, very happy with these lights for the last 10 years or so.  They have a number of modes and colors that you can set by first turning the light on, then cycling the power on them for any of 1-14 times.  I have always wanted to have automation at the pool \(i.e. not dependent on a smarthome product\) that could handle the setting of each of those modes.  Below are the modes and number of times to cycle each one... as well as a couple name changes I took the liberty of making.

| Cycle Count | Original Name | Name in Code | Description |
| ----------- | ------------- | ------------ | ----------- |
| 1 | SAm Mode| SAm | Cycles through white, magenta, blue and green colors (emulates the Pentair SAm® color changing light). |
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


## Wiring Diagram
![Wiring Diagram](https://github.com/jeparkspr/pool-control/blob/main/wiring-diagram.jpg)

## Finished Product
![Finished Product](https://github.com/jeparkspr/pool-control/blob/main/finished-product.jpg)
