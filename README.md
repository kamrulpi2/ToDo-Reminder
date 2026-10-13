# ESP32 + 2.42" SSD1309 OLED (128x64, I2C)

Working examples and a full troubleshooting guide for the blue **M242-12864-4-V2.0** 2.42" OLED module (driver **SSD1309**, 4-pin I2C header: GND, VDD, SCL, SDA) with an **ESP32 DevKit V1**.

<img width="798" height="906" alt="oled-preview (1)" src="https://github.com/user-attachments/assets/50e1c145-8eb2-40b8-bad0-be42e13e35dc" />

> **The #1 problem with this board:** the blank screen / `Nothing found` / wrong address `0x1A` on the I2C scanner.
> **The fix:** short (bridge) the **D1** and **D2** diode pads on the back. See [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md).

## Wiring

| Display pin | ESP32 pin |
|---|---|
| GND | GND |
| VDD | **3V3** (never 5V once D1/D2 are bridged) |
| SCL | GPIO 22 |
| SDA | GPIO 21 |

## Libraries (Arduino IDE -> Library Manager)

- **U8g2** (recommended, works with SSD1309)
- **Adafruit SSD1306** + **Adafruit GFX** (alternative, used by example 03)

Board: `ESP32 Dev Module`, Serial Monitor: `115200`.

## Examples

| Folder | What it does |
|---|---|
| `examples/01_i2c_scanner` | Lists every I2C device found. Use this first. |
| `examples/02_hello_u8g2` | Hello World with U8g2 at address 0x3C. **Known working.** |
| `examples/03_hello_adafruit` | Same with the Adafruit SSD1306 library. |
| `examples/04_address_tester` | Tries 0x3C, 0x3D, 0x1A on screen, one by one. |
| `examples/05_demo_uptime_graph` | Uptime counter, animated bar and a scrolling graph. |

## Full project: BeeClock (clock + bee + todo + alarms + Firebase)

See [projects/bee_clock_todo](projects/bee_clock_todo/README.md): flying-bee clock with moon and falling stars, todo list on the OLED
(works with no internet), phone web app, fast WiFi setup portal, RTOS tasks, Firebase sync. Uses the Adafruit SSD1306 library.
`tools/host_sim` compiles and tests the sketch on a PC.

## Quick start

1. Wire the display as above.
2. Bridge D1 and D2 with solder (see troubleshooting).
3. Power-cycle (unplug and replug USB).
4. Upload `01_i2c_scanner`. You should see `Found device at 0x3C`.
5. Upload `02_hello_u8g2`.

## Board facts (read from the PCB)

- Model: M242-12864-4-V2.0
- Driver IC: SSD1309, 128x64 pixels, interface I2C
- Pads `0X7A` / `0X7B`: I2C address select (0x3C or 0x3D)
- Note printed on the PCB near D2: short D2 if you need the ACK response
- D1 (on SCL) and D2 (on SDA) are series diodes made for 5V-logic boards

## Sources and further reading

- ESP32 + SSD1309 tutorial: https://esp32io.com/tutorials/esp32-ssd1309-oled-display
- Waveshare 2.42" OLED wiki: https://www.waveshare.com/wiki/2.42inch_OLED_Module
- U8g2: https://github.com/olikraus/u8g2
- Forum thread about the D2 / ACK problem: https://forum.arduino.ru/t/oled-2-42-ssd1309-i2c-na-plate-tolko-razyom-i2c-rabotaet-tolko-s-odnoj-bibliotekoj-s-ostalnymi-ne-hochet/10605
- Forum thread about SSD1309 I2C and reset: https://forum.arduino.cc/t/2-42-oled-ssd1309-with-u8glib/425247
