# ESPHome Component: Addressable Light Digital Display With D1 mini (esp8266) 
For my example i use an adressable led for Christmas tree, and D1 mini
I hack a clock (3D Mini Clock LED Digital Wall Clock Desk Clock Electronic Alarm Clock Living Room Wall Clock DIY Kitchen Timer)
* [3D Mini Clock LED Digital Wall Clock](https://fr.aliexpress.com/item/1005006353811631.html)
AND 
20M Dream Color USB 5V LED
* [20M Dream Color USB 5V LED Sting Light Bluetooth Music APP RGBIC Addressable Fairy Lights Birthday Party Garland Christmas Decor](https://fr.aliexpress.com/item/1005006108817683.html)
  
[<img src="images/big_digital_display_by_dawei_zhang_202011.jpg" alt="drawing" height="200"/>](images/big_digital_display_by_dawei_zhang_202011.jpg) 
[<img src="images/small_digital_display_by_dawei_zhang_202201.jpg" alt="drawing" height="200"/>](images/small_digital_display_by_dawei_zhang_202201.jpg) 
[<img src="images/home_assistant_control.jpg" alt="drawing" height="200"/>](images/home_assistant_control.jpg)

This component enables ESPHome to drive an eight-segment display made from FastLED strip with simple configuration.

Comparing to LED matrix and traditional 7-segment display, this display has unique advantages. 
  - It is not constrained by display size or DPI. You can create a large sharp digital display with minimal number of LEDs. It costs much less than using large eight-segment LED modules. 
  - The display can be extended easily. Displaying more digits doesn't increase circuit complexity. Most FastLED can be extended with 3 to 4 wires only in a serial way. No additional ICs or ports required.
  - Higher brightness can be achieved by using more LEDs.
  - Both brightness and colour can be easily controlled.

It's an ESPHome-based general digital display, it can be used for different purposes. For example, it can display date, time, room temperature, noise level etc. Data can also be pulled from Home Assistant.

This project is inspired by the following DIY solutions. 

* [7 segment clock by random1101 for ESPHome
by Alex18881](https://www.thingiverse.com/thing:4689116)
* [DIY BIG CLOCK - USB POWERED NIST SERVER TIME KEEPING by  Ivan Miranda](https://www.youtube.com/watch?v=PixXKK8N_wA)

## DIY advices

### Preparation 

To build a similar digital display, you need the following components:

- LED * N: Any model listed [here](https://esphome.io/components/light/fastled.html#supported-chipsets) is supported, for example `WS2812B`. Bigger display may draw higher power, choose higher votage model can reduce current requirement. 
- A MCU: You need one that ESPHome supported, such as [NodeMCU ESP32](https://esphome.io/devices/nodemcu_esp32.html)
- Power supply: Please make sure the power supply can cope with the maximum load of LEDs and still be able to power the MCU.
- A case: Creating a case from scratch takes time and efforts. If you have a 3D printer, you can download and pringt 3D models from [Thingiverse](https://www.thingiverse.com/) or create one yourself. If you don't have 3D printer, you can buy an existing LED digital clock on market (AliExpress), and modify it. 
- Wires: connect LEDs, MCU and power supply
- A power connector: to connect power supply to your display
- A step down voltage converter: (optional) If you choose 12V LED, you need to convert power to 5V for MCU. 

### Design

The display below uses 30 LEDs (WS2812B) and one D1 mini (esp8266), which can be powered directly via USB 5V 2A power adapter. (5V 1A power is not enough for maximum brightness.)

For each digit to display, LED sequence doesn't matter, because you can configure it later, but it's better to keep the sequence consistent across all digits. 

Besides 7-segment symbols, you can also add special symbols to the display. Currently period `.` and colon `:` are supported.

[<img src="images/small_digital_display_disassembled_by_dawei_zhang_202201.jpg" alt="drawing" height="300"/>](images/small_digital_display_disassembled_by_dawei_zhang_202201.jpg) 

In this case, for each digit, LEDs are connected in the sequence of `BAFEDCG`. The green and blue wires are for data, starting from right to left. 
```
if you want to use "ABCDEFG" in order, you have to follow this order for wiring.
      A
     ---
  F |   | B
     -G-
  E |   | C
     ---
      D
```
### Wiring

Connect the LED in serial way, starting from RIGHT to LEFT of the display. There should be only 3 to 4 wires, 2 of which connect to power supply. Connect the rest to microcontroller. GND is shared by both power supply and microcontroller. 

## Configuration

Read more about external components [here](https://esphome.io/components/external_components.html).

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/daweizhangau/esphome_addressable_light_digital_display
      ref: main
    refresh: 0s
```

### Light component

`id` is required for display component to reference. Make sure correct  `pin`, `num_leds` and `rgb_order` is set. For this time i use HAOS time, see in next code. 

```yaml
light:
  - platform: neopixelbus
    type: GRB
    variant: WS2811
    pin: GPIO2
    num_leds: 30
    id: internal_light_state
```
light:
  - platform: fastled_clockless
  don't work on esp8266 (D1 mini)

### Display component

The display component is exposed as a light to Home Assistant, but it references the FastLED light.

- `platform`: must be `addressable_light_digital_display`
- `id`: ID of the display
- `light_id`: ID of the display light
- `name`: (Optional) Readable name of the display
- `icon`: (Optional) MDI icon ID
- `addressable_light_id`: FastLED light ID defined above
- `restore_mode`: (Optional) Recommended to set it to `ALWAYS_ON` otherwise, you need to turn it on in Home Assistant before it can display anything.
- `default_transition_length`: (Optional) Recommended to set it to `0s`.
- `led_map`: A string that maps LED to the 7 segments (shown below) and special symbols. From left to right, the map string starts from the first LED to the last LED. Each character in this string map to a LED, space is used as a delimiter. `A` to `G` is used to map a LED to corresponding segment of 7-segment, `.` and `:` are used to map corresponding symbols. These characters can be repeated. For example, if 3 LEDs are used to represent Segment B and 3 are used for Segment A, then map string is `BBBAAA`.
- `reverse`: (Optional) Content order. If true, the first led maps to right most character of content string, otherwise, the first LED maps the left most character. True, by default.
- `lambda`: C++ code to formulate display content. 
For more information about print API, please read [printable.h](components/addressable_light_digital_display/printable.h). 
For more information about suported characters, please read [ascii_to_raw.h](components/addressable_light_digital_display/ascii_to_raw.h)
To print time, [`time` component](https://esphome.io/components/time.html) is required.

```yaml
# ---- récupération heure HAOS ----
time:
  - platform: homeassistant
    id: ha_time
display:
  - platform: addressable_light_digital_display
    id: digital_clock
    light_id: digital_display_light
    name: Digital Clock
    addressable_light_id: internal_light_state
    icon: "mdi:clock-digital"
    restore_mode: ALWAYS_ON
    default_transition_length: 0s
    led_map: "BAFEDCG BAFEDCG :: BAFEDCG BAFEDCG"
    reverse: true
    update_interval: 500ms
    lambda: |-
      if (millis() % 1000 < 500)
        it.strftime("%H:%M", id(ha_time).now());
      else
        it.strftime("%H %M", id(ha_time).now());

```
### "BONUS" CHANGE COLOR WITH BINARY SENSOR
I search long time with GPT to change color when my "WC01" light is on or off. 
Changing color was very hard because D1 mini don't support a lot of variable, set_color/set_rgb or other don't work.
So the idea is to create a binary sensor and change color of "light_id: digital_display_light"
My sensor is on another esp32, you can use all entity on Homeassistant.

```yaml
# ---- Binary sensor pour WC01 ----
binary_sensor:
  - platform: homeassistant
    id: wc01_led_state 
    entity_id: light.wc01_led
    on_press:
      - light.turn_on:
          id: digital_display_light
          green: 100%
          red: 0%
          blue: 0%
    on_release:
      - light.turn_on:
          id: digital_display_light
          green: 0%
          red: 0%
          blue: 100%
```
### ALL CODE 
```yaml
captive_portal:
# ---- récupération heure HAOS ----
time:
  - platform: homeassistant
    id: ha_time

light:
  - platform: neopixelbus
    type: GRB
    variant: WS2811
    pin: GPIO2
    num_leds: 30
#    name: "Horloge"
    id: internal_light_state
#    color_correct: [0%,0%,100%] #// don't set color here, changing color don"t work if you set here.

display:
  - platform: addressable_light_digital_display
    id: digital_clock
    light_id: digital_display_light
    name: Digital Clock
    addressable_light_id: internal_light_state
    icon: "mdi:clock-digital"
    restore_mode: ALWAYS_ON
    default_transition_length: 0s
    led_map: "BAFEDCG BAFEDCG :: BAFEDCG BAFEDCG"
    reverse: true
    update_interval: 500ms
    lambda: |-
      if (millis() % 1000 < 500)
        it.strftime("%H:%M", id(ha_time).now());
      else
        it.strftime("%H %M", id(ha_time).now());

# ---- Binary sensor pour WC01 ----
binary_sensor:
  - platform: homeassistant
    id: wc01_led_state
    entity_id: light.wc01_led
    on_press:
      - light.turn_on:
          id: digital_display_light
          green: 100%
          red: 0%
          blue: 0%
    on_release:
      - light.turn_on:
          id: digital_display_light
          green: 0%
          red: 0%
          blue: 100%
```

