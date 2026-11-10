# IoT Environmental Monitoring System – Arduino Nano PCB

A multi-sensor environmental monitoring hardware project built around an **Arduino Nano**. The custom PCB brings together temperature, humidity, pressure, light, motion, smoke, and gas-sensing modules, along with an OLED display and a buzzer output.

The hardware schematic, PCB layout, and 3D board preview are designed in **KiCad**. This repository can be used as the project documentation and as a starting point for PCB fabrication, firmware development, assembly, and testing.

> **Project status:** The schematic and PCB layout/3D visualisation are the current design deliverables. Update this README with fabrication, assembly, firmware, and test results after those stages have been completed.

---

## Table of contents

- [Project overview](#project-overview)
- [Features](#features)
- [Hardware components](#hardware-components)
- [System architecture](#system-architecture)
- [Signal and interface list](#signal-and-interface-list)
- [Power and wiring notes](#power-and-wiring-notes)
- [PCB design](#pcb-design)
- [Getting started](#getting-started)
- [Firmware development](#firmware-development)
- [Calibration and testing](#calibration-and-testing)
- [Safety notes](#safety-notes)
- [Repository structure](#repository-structure)
- [Future improvements](#future-improvements)
- [Contributing](#contributing)
- [License](#license)
- [Author](#author)

---

## Project overview

The goal of this project is to create a single PCB that connects an Arduino Nano to several commonly used sensor modules. The Arduino can collect readings from the connected sensors, process the measurements, display information on an OLED, and activate a buzzer when firmware-defined conditions are met.

The board is designed with connectors for the individual modules so that they can be connected, replaced, or tested separately. The PCB layout and 3D preview help check connector placement and the overall board arrangement before fabrication.

### Monitoring modules

| Module | Intended measurement or function | Interface |
|---|---|---|
| DHT22 | Ambient temperature and relative humidity | Digital, single-wire style interface |
| MQ-135 | General air-quality-related gas response | Analog output from sensor module |
| MQ-2 | Smoke and combustible-gas response | Analog output from sensor module |
| BMP280 | Atmospheric pressure and temperature | I²C |
| LDR module | Relative light level | Analog output from module |
| HC-SR501 PIR | Motion detection | Digital output |
| DS18B20 | Contact temperature measurement | 1-Wire |
| OLED display | Displays readings and system information | I²C |
| Buzzer | Audible status or alert output | Digital control |

The MQ-series sensors are broad-response sensors. Their readings require suitable warm-up, calibration, and interpretation; they should not be treated as precise gas-concentration measurements without a validated calibration method.

---

## Features

- Arduino Nano as the central controller.
- Multiple sensor-module connectors on one PCB.
- Separate sensor connections for temperature, humidity, pressure, light, motion, smoke, and gas-response measurements.
- I²C connection for the BMP280 and OLED display.
- Dedicated buzzer control connection.
- PCB schematic and board layout created in KiCad.
- 3D PCB visualisation for checking component and connector placement.
- Modular connectors to simplify sensor replacement and troubleshooting.

---

## Hardware components

| No. | Component | Quantity | Purpose |
|---:|---|---:|---|
| 1 | Arduino Nano | 1 | Main microcontroller |
| 2 | DHT22 module | 1 | Temperature and humidity sensing |
| 3 | MQ-135 module | 1 | General air-quality-related gas response |
| 4 | MQ-2 module | 1 | Smoke/combustible-gas response |
| 5 | BMP280 module | 1 | Pressure and temperature sensing |
| 6 | LDR light-sensor module | 1 | Light-level sensing |
| 7 | HC-SR501 PIR module | 1 | Motion detection |
| 8 | DS18B20 sensor | 1 | Contact temperature sensing |
| 9 | OLED display module | 1 | Local display |
| 10 | Buzzer | 1 | Audible alert/status output |
| 11 | 10 kΩ resistor | 1 | DHT22 data-line pull-up, as shown in the schematic |
| 12 | 4.7 kΩ resistor | 1 | DS18B20 data-line pull-up, as shown in the schematic |
| 13 | PCB headers/connectors | As required | Sensor and module connections |

**Note:** Confirm the exact module variants, connector pin order, resistor values, footprints, and quantities against the KiCad schematic and BOM before ordering parts or fabricating the PCB. Some sensor modules are sold in multiple pinout and voltage variants.

---

## System architecture

```text
                     +----------------------+
                     |     Arduino Nano     |
                     |   Main Controller    |
                     +----------+-----------+
                                |
       +------------------------+-------------------------+
       |             |          |          |               |
       v             v          v          v               v
    DHT22         DS18B20     HC-SR501   LDR module     MQ-135 / MQ-2
  Temp/RH         Temp        Motion     Light level   Gas/smoke response
       |             |          |          |               |
       +-------------+----------+----------+---------------+
                                |
                         I²C bus / outputs
                                |
                       +--------+--------+
                       |                 |
                       v                 v
                   BMP280          OLED display
                 Pressure/temp     Readout/status
                                |
                                v
                             Buzzer
                          Audio alert
```

The diagram is a functional overview, not a substitute for the actual schematic. Use the KiCad schematic as the source of truth for electrical connections and pin assignments.

---

## Signal and interface list

The schematic uses the following signal names. Match the firmware to the final net-to-pin assignments in the KiCad schematic.

| Schematic net name | Connected module | Purpose |
|---|---|---|
| `DHT_DATA` | DHT22 | Temperature/humidity data |
| `DS18B20_DATA` | DS18B20 | 1-Wire temperature data |
| `PIR_OUT` | HC-SR501 | Digital motion-detection output |
| `BUZZER_CTRL` | Buzzer | Buzzer control |
| `MQ135_AO` | MQ-135 | Analog sensor output |
| `MQ2_AO` | MQ-2 | Analog sensor output |
| `LDR_AO` | LDR module | Analog light-level output |
| `I2C_SDA` | BMP280 and OLED | I²C data line |
| `I2C_SCL` | BMP280 and OLED | I²C clock line |

**Important:** The table intentionally lists schematic net names rather than guessed Arduino pin numbers. Check the schematic before writing firmware or connecting a module. Confirm that the physical connector pin order is `VCC`, `GND`, and signal (or the order actually printed in the schematic); never assume all modules use the same order.

---

## Power and wiring notes

- Follow the power-net labels in the KiCad schematic. The design includes **+5 V** and **+3.3 V** connections; do not interchange them.
- Connect all module grounds to the board's common ground as shown in the schematic.
- Check the voltage requirements and logic levels of each exact breakout/module before connecting it.
- The BMP280 IC itself is a 3.3 V device. Some breakout boards include a regulator and/or level shifting, while others do not. Verify the specific module before connecting it to a 5 V Arduino Nano I²C bus. Add suitable I²C level shifting when required.
- MQ sensor modules contain heater elements and can draw considerably more current than simple digital sensors. Provide a suitable regulated supply with adequate current capacity; do not assume the Arduino Nano's onboard regulator can power every module.
- Check polarity and connector pin order before applying power.
- Keep sensor power and signal wiring within the voltage/current limits of the modules and the Arduino Nano.

---

## PCB design

The PCB was designed in **KiCad**. The design workflow includes:

1. Creating the circuit schematic and assigning net labels.
2. Assigning footprints to the Arduino Nano and module connectors.
3. Placing the connectors and routing the PCB tracks.
4. Checking the board outline, clearances, and connector positions.
5. Reviewing the board in KiCad's 3D Viewer.

### Design files

Add the KiCad project files to the repository and update these paths to match your actual filenames:

- `hardware/` – KiCad project files.
- `hardware/*.kicad_pro` – project settings.
- `hardware/*.kicad_sch` – circuit schematic.
- `hardware/*.kicad_pcb` – PCB layout.
- `hardware/*.kicad_dru` – design rules, if used.
- `hardware/fabrication/` – Gerber, drill, and fabrication outputs, once generated.

Before fabrication, run the Electrical Rules Checker (ERC) and Design Rules Checker (DRC), review all warnings, and verify connector footprints and pin numbering against the physical modules.

---

## Getting started

### 1. Install KiCad

Install a recent stable version of KiCad from the official website:

https://www.kicad.org/download/

### 2. Clone or download this repository

```bash
git clone https://github.com/<your-username>/<your-repository-name>.git
cd <your-repository-name>
```

Replace the placeholder URL with your actual GitHub repository URL.

### 3. Open the PCB design

1. Open KiCad.
2. Open the `.kicad_pro` project file from the `hardware/` folder.
3. Open the schematic editor to inspect circuit connections.
4. Open the PCB Editor to inspect component placement and routing.
5. Open the 3D Viewer to inspect the board's 3D representation.

### 4. Check before fabrication

- Run ERC on the schematic.
- Run DRC on the PCB.
- Confirm all footprints and connector pin numbering.
- Check board dimensions, mounting requirements, clearances, and copper widths.
- Verify supply rails and current capacity, especially for the MQ sensor modules.
- Generate Gerber and drill files only after the design checks are complete.

---

## Firmware development

The Arduino Nano is intended to read the sensor modules and manage the OLED and buzzer. Add the firmware source code to a `firmware/` directory when it is ready.

A typical firmware structure could look like this:

```text
firmware/
└── environmental_monitor/
    ├── environmental_monitor.ino
    └── README.md
```

### Suggested firmware behaviour

1. Initialise the sensor modules and OLED display.
2. Read temperature and humidity from the DHT22.
3. Read temperature from the DS18B20.
4. Read pressure and temperature from the BMP280 over I²C.
5. Read analog values from the MQ-135, MQ-2, and LDR modules.
6. Read the digital output from the HC-SR501 PIR module.
7. Display available readings and motion/status information on the OLED.
8. Activate the buzzer according to configurable thresholds or events.

### Possible Arduino libraries

Choose library versions compatible with your installed Arduino IDE and the exact sensor modules:

- Adafruit BMP280 Library (BMP280)
- Adafruit Unified Sensor (when required by the selected Adafruit library)
- DHT sensor library (DHT22)
- OneWire (DS18B20)
- DallasTemperature (DS18B20)
- Adafruit SSD1306 or U8g2 (OLED, depending on the display controller)

The MQ-135 and MQ-2 analog outputs can be read using the Arduino analog-input functions. Converting raw readings into meaningful gas estimates requires sensor-specific calibration and a suitable measurement model.

---

## Calibration and testing

Test one module at a time before testing the complete system.

| Test | What to verify |
|---|---|
| Power rails | Measure the expected +5 V and +3.3 V rails before inserting modules |
| Arduino Nano | Confirm that the Nano powers up and can upload a basic test sketch |
| DHT22 | Compare temperature and humidity readings with a reference instrument |
| DS18B20 | Check the 1-Wire connection and compare readings with a reference thermometer |
| BMP280 | Confirm I²C detection and check pressure/temperature readings |
| OLED | Confirm display initialisation and readable output |
| LDR | Check that the analog reading changes between bright and dark conditions |
| PIR | Check motion detection and allow for the module's settling/initialisation time |
| MQ-135 / MQ-2 | Allow the required preheating period and follow the module datasheet's calibration guidance |
| Buzzer | Confirm that the control signal activates the buzzer as intended |
| Complete board | Check for unexpected heating, unstable readings, resets, or power-supply problems |

Record the test date, supply voltage, sensor/module variant, calibration method, and observed results. Do not publish unverified accuracy figures.

---

## Safety notes

- This is an educational electronics project, not a certified environmental, industrial, or life-safety instrument.
- MQ-2 and MQ-135 modules are not substitutes for certified smoke, fire, carbon-monoxide, or hazardous-gas alarms.
- Do not use this project as the only protection against fire, gas leaks, or unsafe air conditions.
- MQ sensor readings are affected by warm-up time, temperature, humidity, sensor age, and other gases. They are not automatically equivalent to calibrated ppm values.
- Use a properly rated power supply and avoid short circuits or reversed polarity.
- Disconnect power before changing wiring or modifying the PCB.

---

## Repository structure

A suggested GitHub structure is:

```text
IoT-Environmental-Monitoring-PCB/
│
├── hardware/
│   ├── schematic/          # KiCad schematic files
│   ├── pcb/                # KiCad PCB layout files
│   ├── libraries/          # Project-specific symbols/footprints, if any
│   ├── fabrication/        # Gerber and drill files (after design validation)
│   └── 3d-models/           # Additional 3D models, if required
│
├── firmware/
│   └── environmental_monitor/
│       └── environmental_monitor.ino
│
├── images/
│   ├── schematic.png
│   ├── pcb-layout.png
│   └── pcb-3d-view.png
│
├── docs/
│   ├── bill-of-materials.md
│   └── test-results.md
│
├── README.md
└── LICENSE
```

Keep only folders and files that actually exist in your repository. Add your schematic, PCB layout, and 3D-view screenshots to `images/` and use the following Markdown to display them (update filenames as needed):

```md
## Project images

### Circuit schematic
![Circuit schematic](images/schematic.png)

### PCB layout
![PCB layout](images/pcb-layout.png)

### PCB 3D view
![PCB 3D view](images/pcb-3d-view.png)
```

---

## Future improvements

- Complete PCB fabrication and physical assembly.
- Develop and upload the Arduino Nano firmware.
- Add a structured OLED screen layout for sensor readings and system status.
- Implement configurable buzzer alerts.
- Calibrate the MQ sensor modules using an appropriate documented procedure.
- Perform individual sensor tests and complete-board validation.
- Add an enclosure and mounting provisions, if required.
- Add data logging or wireless connectivity in a future version, if required.

---

## Contributing

Suggestions, bug reports, and improvements are welcome. To contribute:

1. Fork the repository.
2. Create a feature branch.
3. Make your changes and document any hardware or firmware updates.
4. Test the changes where possible.
5. Submit a pull request with a clear description and relevant screenshots.

For hardware changes, include updated KiCad files and note any changes to component values, footprints, connector pinouts, or power requirements.

---

## License

Choose and add a license before sharing or accepting contributions. For example, hardware designs may use the **CERN Open Hardware Licence**, while firmware/software may use an appropriate open-source software licence. Do not claim a license until its license file has been added to this repository.

---

## Author

**Hari Prasad L S**  
EEE Student | Embedded Systems | IoT | PCB Design

- GitHub: https://github.com/lshariprasad

---

⭐ If you find this project useful, consider starring the repository and sharing your feedback!
