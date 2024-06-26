# <img src="assets/img/logo.svg" width="48" width="48" /> HA PC Switch

**Automate power on your PC using [ESPHome](https://github.com/esphome/esphome) or [Tasmota](https://github.com/arendst/Tasmota), and integrated into [Home Assistant](https://www.home-assistant.io/).**

---

**`STATUS`**: This project is in ideation stage.  Come back soon for progress.

**`LAST UPDATE`**: 7 June 2024

---

## Origins

Looking for a way to remotely start-up and shut-down a PC from Home Assistant.  Also need operational status (sensors) to drive automations.

### June 2024 progress

In a case of [Multiple Discovery](https://en.wikipedia.org/wiki/multiple_discovery), it turns out that this idea already has commercial solutions for sale. I've found a few variants on AliExpress:

1. [Computer Remote Switch,Wifi Smart PC Start Boot Card,Startup Card Work with Apple Homekit SIRI](https://www.aliexpress.com/item/1005005378143852.html) - *This one I really like and have bought myself, as it's written specifically for the Apple Homekit eco-system that I use.I also like that it uses the new ESPC2-12 module.*

2. [TISHRIC Wifi PC Power Switch Computer Remote Boot Startup Card Computer Restart Switch Timing Boot Device Work With Alexa Google](https://www.aliexpress.com/item/1005004861684220.html) - *I really like the PCI bracket plate of this one. Depends on Tuya though* :( *.*

3. [eWeLink Computer Remote Boot Card Remote Control Wireless WIFI Switch Relay Module For Computer Work with Google Home and Alexa](https://www.aliexpress.com/item/1005001727371911.html) - *I think this one is one of the first modules developed.  Also relies on Tuya though* :( *.*

**However** - none of the above are open source - so we don't have the ability to modify the firmware ... and to add to that, most rely on the Tuya eco-system - so not viable for off-line use, and adds the "cloud" factor to it's use.

So I'm progressing with building ... EE design is done, but I need to change focus to building the firmware.

<img src="assets/img/rev_b_frnt.png" width="33%">
&nbsp;
<img src="assets/img/rev_b_back.png" width="33%">

## Ideas

### Other People's Projects

* Zvonko Bockaj *@Hackster* - [WEMOS ESP8266 Remote PC Switch](https://www.hackster.io/zvonko-bockaj/wemos-esp8266-remote-pc-switch-062c7a) `// this one makes most sense`
* `ajfriesen` - [The pc-switch](https://www.ajfriesen.com/pc-switch/) `// great source for ESPHome YAML example`
* `Erriez` *@Github* - [ESPHomePCPowerControlHomeAssistant](https://github.com/Erriez/ESPHomePCPowerControlHomeAssistant/) `// ditto`

&

* `SilverFire` *@Github* - [esp8266-pc-power-control](https://github.com/SilverFire/esp8266-pc-power-control/)
* Lerk - [Turning a PC on and off using an ESP8266](https://lerks.blog/p/turning-a-pc-on-and-off-using-an-esp)

## Specifications (as a wish list)

### Inputs

1. ~~Power Standby (from PSU pin 9 {purple} / +5V SB (Standby)) - *as source of operating +5V power*~~
2. ~~Power Good (from PSU, pin 8 {grey} / PG (Pwr Good))~~
3. ~~Power Supply ON (from PSU, pins 21,22 {red} )~~
4. ~~PC ON (from Case LED)~~
5. ~~Case / MLB POWER Button~~
6. ~~Case / MLB Reset Button~~

* ~~GND (2x sources from PSU & MLB)~~

[June '24 update] - dropping the need for inputs.  Only one is needed at this stage .. i.e. is the PC ON or not.  Also removing the need for PSU monitoring, **and** it's source of power.  Turns out that PCIe bus supplies a 3.3V "Aux" line (a.k.a. "Standby" power), so we can just tap of the PCIe slot.

### Sensors (exposed to HA)

* ~~Power Good (from PSU, pin 8 {grey, *measure `true` when +5V*})~~
* ~~Power (Supply) ON (from PSU, pin 16 {green, *measure `false` when +5V* } / PS-ON (Pwr Sply switch))~~
    * ~~*This is the pin that need to be grounded for the PSU to switch on, so needs careful testing to make sure one does not have false ON states.*~~
* ~~(PC) Power ON (from MLB, case LED) {*measure `true` when +5V*}~~
* ~~Optional: via D1 Mini hats~~
    * ~~*(e.g. Case temperature)*~~
    * ~~expose D1 and D2 pins ...~~
    * ~~... or as I2C bus~~


### Outputs
* Power "relay" *(See note about "relay"'s in ideas below)*
* Reset "relay"
* ~~Case LED (as passthrough)~~
* ~~Optional: via D1 Mini hats~~
    ~~*(e.g. Contact Relay, as power to RGB LED's switch)*~~

[June '24 Update] - Solid state / MOSFET relays are still in.  Addon outputs are however dropped.  But... I2C is exposed as a single QWICC connector in case one wants to extend.

##### Terms used above
* **PSU** = Power Supply (Unit) <br/>
* **MLB** = Mail (Logic) Board / PC Motherboard 

## Ideas

* ~~"Sensor" should be isolated from circuit with an Op-amp Comparator](https://www.electronics-tutorials.ws/opamp/op-amp-comparator.html) or [Level Shifter](https://www.sparkfun.com/products/12009) circuit.  *(Inputs will be 5V, whilst the ESPxx works on 3.3v.)*~~

* "Relay" should be an [Optocoupler](https://www.electronics-tutorials.ws/blog/optocoupler.html)
    * ~~Do **<u>not</u>** use a transistor, nor a mechanical relay~~

    * Some PC's have "Positive" switches, others have "Negative" switches.  Accounting for this is too complex *(i.e. prone to installer error)*, so would it be better to use a ~~*relay* *(e.g. Omron G6K [SMD @ &#xB1;$7], or Songle `SRD-3VDC-SL-C` / Omrom G5LE [THD @ &#xB1;$0.36&#xA2;] )*~~.

        The issue here is that the MLB *(PC Motherboard)* may be using a low-sensing circuit, or a high-sensing circuit - and this PCB must account for both!

        <img src="assets/img/high_vs_low_sensing.png" /><br/>

        All we get access to is the points that are the pins of the switch.

    * Solution looks to be a TIP3123 opto-MOSFET, thanks to [Andy aka](https://electronics.stackexchange.com/questions/548380/how-is-it-possible-to-replace-a-relay-with-transistors).

* &#x26A0; "Relay" should be driven by a "Delay OFF type" circuit, and 2 timings must be provided
    * 300ms delay from ON back to OFF to simulate a "press".
    * 5.5s delay on ON, then back to OFF to simulate a "long press".

        <img src="assets/img/one-shot-normally-open.png" width="67%" height="67%" /><br/>

    3 possible ways to do this:

    1. EE solution with caps and transistors - *not really viable since because of its variable performance and unchangeable nature. (See [here](https://www.homemade-circuits.com/simple-delay-timer-circuits-explained/))*

    2. If Software solution (i.e. on ESPHome or Tasmota), then how do we create a **"One-Shot" Normally-Open** switch?

        * ESPHome, can do this with with a [`on_turn_on` trigger](https://esphome.io/components/switch/gpio.html#momentary-switch).
        
        * Tasmota, can do this with "*Rules*" on the device.

        * Home Assistant with an automation trigger using an "`action`" > "`sequence`".

        But, what if the user forgets to "modify" the default f/w or build the HA automation required?  The device ***<u>must</u>*** handle the one-shot functionally itself.

        &#x26A0; Custom f/w (or fork of) must be built for timed "one-shot" / "delay off" functionality.

    3. Usa a second MCU with a timing circuit - e.g. ~~ATtiny85 or **STM8S001J3**~~ or **STM32C0**.

        * May be useful as a feedback "sensor".
        * Don't forget to provide programming headers.

* ~~Tap into the PSU ATX Cable for power and PSU inputs~~
    * ~~Std ATX~~ <br/>
        <img src="assets/img/atx_pinout.png" height="50%" width="50%" />
    * ~~Use a ATX power extender to "tap" into correct cables~~ <br/>
        <img src="assets/img/atx_adapter.png" height="33.3%" width="33.3%" />

* ~~Use the WEMOS D1 Mini form-factor and pin-outs, but prototype on:~~
    * ~~[WEMOS D1 Mini Pro](https://www.aliexpress.com/item/1005006109635545.html) (with external antenna) <br/>~~
      <img src="assets/img/d1-mini-pro.png" height="33.3%" width="33.3%" />
    * ~~[LOLIN ESP32 S2 Mini](https://www.aliexpress.com/item/1005006157693055.html)~~ <br/>
      <img src="assets/img/esp32-s2-mini .png" height="33.3%" width="33.3%" />

* [May '24 Update] ~~Also add a~~ footprint for the **ESP-12F**/ESP-12S module
    * Cheaper, but needs external programmer (add headers for that).

      <img src="assets/img/esp-12f.png" width="25%" height="25%" />

    * [Update '24 Update] Both the **ESP-07** and the newer **ESPC2-12** modules have the same footprint and pinout, adding far more versatility.

* ~~Build the PCB in a way that the unit can attach to a **PCI Slot Bracket Cover**, specifically build for:~~
    * ~~PCI Slot Fan Mount Rack for video card for 90mm and 120mm fans~~ <br/>
      <img src="assets/img/slot-fan-bracket.png" height="50%" width="50%" />
    * ~~PCI Slot for 2.5inch hard drive, rear panel mount~~ <br/>
      <img src="assets/img/slot-hdd-caddy.png" height="50%" width="50%" />

    Will allow for drilling holes for external antenna.

* [June '24 Update] Build to [PCIe V2 specification](specs/pci-express-card-electromechanical-specification-revision-2.pdf).  To fit standard PCI Bracket plates:
  
  * [Example 1](https://www.aliexpress.com/item/1005005341638856.html)
  * [Example 2](https://www.aliexpress.com/item/1005004003916975.html)

---

Made with &#x1F499; by [<img src="assets/img/vino-face.svg" width="16" height="16" />](https://github.com/vinorodrigues).
