# <img src="assets/img/logo.svg" width="48" width="48" /> Home Assistant PC Switch

**Control your PC from Home Assistant using [ESPHome](https://github.com/esphome/esphome) or [Tasmota](https://github.com/arendst/Tasmota), and integrated into [Home Assistant](https://www.home-assistant.io/).**

---

**`STATUS`**: This project is in ideation stage.  Come back soon for progress.

---

> *"A good scientist is a person with original ideas. A good engineer is a person who makes a design that works with as few original ideas as possible. There are no prima donnas in engineering."* -- Freeman Dyson

## Origins

Looking for a way to remotely start-up and shut-down a PC from Home Assistant.  Also need operational status (sensors) to drive automations.

## Ideas

### Other People's Projects

* `SilverFire` *@Github* - [esp8266-pc-power-control](https://github.com/SilverFire/esp8266-pc-power-control/)
* Lerk - [Turning a PC on and off using an ESP8266](https://lerks.blog/p/turning-a-pc-on-and-off-using-an-esp)
* Zvonko Bockaj *@Hackster* - [WEMOS ESP8266 Remote PC Switch](https://www.hackster.io/zvonko-bockaj/wemos-esp8266-remote-pc-switch-062c7a)
    <!--
    * &#x1F44D; Based on WEMOS D1 mini module
    * &#x1F44D; ON state sensor (using case header LED)
    * &#x1F44E; NPN Transistor based switch
    * &#x1F44E; Custom Arduino code, generating custom MQTT
    -->
* `ajfriesen` - [The pc-switch](https://www.ajfriesen.com/pc-switch/)
    <!--
    * &#x1F44D; Based on WEMOS D1 mini module
    * &#x1F44D; Photocoupler based based switch
    * &#x1F44D; ESPHome `.yaml` file provided.
    -->
* `Erriez` *@Github* - [ESPHomePCPowerControlHomeAssistant](https://github.com/Erriez/ESPHomePCPowerControlHomeAssistant/)
    <!--
    * &#x1F44E; Based on NodeMCU module
    * &#x1F44E; NPN Amplifier Transistor based switch
    * &#x1F44E; No passthrough for power and reset switch
    * &#x1F44D; ESPHome `.yaml` file provided.
    -->

## Specifications (as a wish list)

### Inputs

1. Power Standby (from PSU pin 9 / +5V SB (Standby)) - *as source of operating +5V power*
2. Power Good (from PSU, pin 8 / PG (Pwr Good))
3. Power Supply ON (from PSU)
4. PC ON (form Case LED)
5. Case / MLB POWER Button
6. Case / MLB Reset Button
7. GND (2x sources from PSU & MLB)

### Sensors (exposed to HA)

* Power Good (from PSU)
* Power (Supply) ON (from PSU, pin 16 / PS-ON (Pwr Sply On))
* (PC) Power ON (from MLB, case LED)

### Outputs
* Power "relay" *(See note about "relay"'s in ideas below)*
* Reset "relay"
* Case LED (as passthrough)

##### Terms used above
* **PSU** = Power Supply (Unit) <br/>
* **MLB** = Mail (Logic) Board / PC Motherboard 

## Ideas

* "Relay" should be an [Optocoupler](https://www.electronics-tutorials.ws/blog/optocoupler.html), or an [Op-amp Comparator](https://www.electronics-tutorials.ws/opamp/op-amp-comparator.html)
    * Do **<u>not</u>** use a transistor, nor a mechanical relay

* "Relay" delay should not be software based
    * 300ms delay from ON back to OFF to simulate a "press".
    * 5.5s delay on ON, then back to OFF to simulate a "long press".

* Tap into the PSU ATX Cable for power and PSU inputs
    * Std ATX <br/>
        <img src="assets/img/atx_pinout.png" height="33.3%" width="33.3%"/>
    * Use a ATX power extender to "tap" into correct cables <br/>
        <img src="assets/img/atx_adapter.png" height="33.3%" width="33.3%"/>

* Use the WEMOS D1 Mini form-factor and pin-outs, but prototype on:
    * [WEMOS D1 Mini Pro](https://www.aliexpress.com/item/1005006109635545.html) (with external antenna) <br/>
      <img src="assets/img/d1-mini-pro.png" height="33.3%" width="33.3%" />
    * [LOLIN ESP32 S2 Mini](https://www.aliexpress.com/item/1005006157693055.html) <br/>
      <img src="assets/img/esp32-s2-mini .png" height="33.3%" width="33.3%" />

* Build the PCB in a way that the unit can attach to a **PCI Slot Bracket Cover**, specifically build for:
    * PCI Slot Fan Mount Rack for video card for 90mm and 120mm fans <br/>
      <img src="assets/img/slot-fan-bracket.png" height="33.3%" width="33.3%">
    * PCI Slot for 2.5inch hard drive, rear panel mount <br/>
      <img src="assets/img/slot-hdd-caddy.png" height="33.3%" width="33.3%">

    Will allow for drilling holes for external antenna.
