# Bill of Materials — Cyclone III EP3C16F484C6N
## 50 Advanced FPGA/VHDL Projects | Hardware-Design-Engineer-VHDL-Portfolio

**Platform:** Intel/Altera Cyclone III EP3C16F484C6N + ESP32 + Raspberry Pi 3B / Pi 5  
**Toolchain:** Quartus Prime Lite, ModelSim-Altera, LTSpice, KiCad  
**Estimated Total:** ≈ R 14 800 (excl. VAT, shipping, PCB fabrication)  
**Local suppliers:** RS Components ZA · Mantech · Communica · AliExpress

---

## 1. Analog Front-End & Signal Conditioning
*Projects 16 (TIA Photodiode), 17 (Laser APC), 18 (Lock-In Amp), 19 (VGA/AGC)*

| # | Component | Part Number | Package | Qty | Est. Unit | Est. Total | Supplier |
|---|---|---|---|---|---|---|---|
| 1 | Wideband Op-Amp 1.6 GHz GBW | OPA657N | DIP-8 | 3 | R 140 | R 420 | RS Components ZA |
| 2 | Precision Low-Noise Dual Op-Amp | OPA2134UA | SOIC-8 | 4 | R 70 | R 280 | RS Components ZA |
| 3 | Variable Gain Amplifier 45 dB (SPI) | AD8338ACPZ | LFCSP-16 | 2 | R 190 | R 380 | Mouser / RS |
| 4 | CMOS Quad Analog Switch RON 5Ω | ADG733BRUZ | TSSOP-16 | 2 | R 80 | R 160 | RS Components ZA |
| 5 | PIN Photodiode Si 850 nm | BPW34 | TO-92 | 5 | R 18 | R 90 | Communica |
| 6 | Laser Diode 650 nm 5 mW | CQL806J | TO-18 | 3 | R 50 | R 150 | AliExpress |

**Subtotal: R 1 480**

---

## 2. High-Speed ADC & DAC
*Projects 7 (ADC Bridge), 15 (Programmable Voltage Ref), 17 (Laser APC), 27–30 (FIR/FFT/DSO)*

| # | Component | Part Number | Package | Qty | Est. Unit | Est. Total | Supplier |
|---|---|---|---|---|---|---|---|
| 7 | 12-bit 65 MSPS Parallel ADC | AD9226ARSZ | SSOP-28 | 2 | R 325 | R 650 | Mouser |
| 8 | 16-bit Precision Bipolar DAC ±10V | AD5762BREZ | TSSOP-24 | 2 | R 360 | R 720 | RS Components ZA |
| 9 | 12-bit SPI DAC 20 MHz | DAC8811IDRG4 | SOIC-8 | 3 | R 120 | R 360 | RS Components ZA |
| 10 | 24-bit Sigma-Delta ADC 4-ch (I²C) | ADS1115IDGSR | SOIC-8 | 4 | R 70 | R 280 | Communica |
| 11 | Bidirectional Power Monitor 36V 16-bit | INA226AIDT | SOT-23-8 | 4 | R 55 | R 220 | Communica |

**Subtotal: R 2 230**

---

## 3. Mil-Spec Power Supply & Protection
*Projects 13 (28V→5V Buck), 14 (Power Sequencer), 15 (Programmable Reference)*

| # | Component | Part Number | Package | Qty | Est. Unit | Est. Total | Supplier |
|---|---|---|---|---|---|---|---|
| 12 | Half-Bridge Gate Driver Bootstrap | IR2110PBF | DIP-14 | 3 | R 60 | R 180 | RS Components ZA |
| 13 | N-Ch MOSFET 100V 57A RDS 23mΩ | IRF3710PBF | TO-220 | 6 | R 40 | R 240 | Communica |
| 14 | Power Inductor 10µH 15A Shielded | SRP1265A-100M | SMD | 4 | R 80 | R 320 | RS Components ZA |
| 15 | Common-Mode EMI Choke 700µH | ACM7060-701-2PL | SMD | 4 | R 70 | R 280 | RS Components ZA |
| 16 | TVS Diode 28V 600W Unidirectional | SMBJ28A | SMB | 10 | R 10 | R 100 | Communica |
| 17 | NTC Thermistor 10kΩ B=3977K | NTCLE100E3103JB0 | 0603 | 5 | R 12 | R 60 | RS Components ZA |
| 18 | Precision Shunt Resistor 0.01Ω 1% 3W | WSBS20001L000FEB | TO-263 | 6 | R 30 | R 180 | RS Components ZA |
| 19 | Schottky Diode 60V 5A (bootstrap) | SS54 | SMA | 10 | R 8 | R 80 | Communica |
| 20 | 28V DC Terminal Block 5mm pitch | Generic | TH | 5 | R 16 | R 80 | Communica |

**Subtotal: R 1 520**

---

## 4. Communication Interface ICs
*Projects 8 (CAN), 9 (GigE MAC), 10 (Aurora/LVDS), 31–34 (HDLC, 1553B, ARINC 429)*

| # | Component | Part Number | Package | Qty | Est. Unit | Est. Total | Supplier |
|---|---|---|---|---|---|---|---|
| 21 | CAN Bus Transceiver 5 Mbps | MCP2551-I/P | DIP-8 | 4 | R 30 | R 120 | Communica |
| 22 | RS-422/485 Transceiver ±15kV ESD | MAX490ECPD+ | DIP-8 | 6 | R 30 | R 180 | Communica |
| 23 | MIL-STD-1553B Bus Transceiver | UT63M125V | CERDIP-16 | 2 | R 700 | R 1 400 | RS Components ZA |
| 24 | ARINC 429 Dual Rx + Tx | HI-3593PQI | SOIC-28 | 2 | R 490 | R 980 | Mouser |
| 25 | 100 Mbps Ethernet PHY MII/RMII | KSZ8091RNBA | TQFP-32 | 2 | R 120 | R 240 | RS Components ZA |
| 26 | Quad LVDS Driver | SN65LVDS31QDRQ1 | SOIC-16 | 4 | R 70 | R 280 | RS Components ZA |
| 27 | Quad LVDS Receiver | SN65LVDS32BDRQ1 | SOIC-16 | 4 | R 70 | R 280 | RS Components ZA |
| 28 | USB-UART Bridge 3.3V | CP2102N-A02-GQFN28 | QFN-28 | 4 | R 50 | R 200 | Communica |
| 29 | RJ45 + Integrated Magjack | HR911105A | TH | 4 | R 30 | R 120 | Communica |
| 30 | DB9 Connector Male/Female pair | Generic | TH | 8 | R 10 | R 80 | Communica |

**Subtotal: R 3 880**

---

## 5. Memory & Storage
*Projects 6 (SDRAM Controller), 30 (DSO waveform storage), 36 (Data Logger), 42 (Remote Update)*

| # | Component | Part Number | Package | Qty | Est. Unit | Est. Total | Supplier |
|---|---|---|---|---|---|---|---|
| 31 | SDRAM 64Mbit 16-bit 143 MHz | IS42S16400J-7TLI | TSOP-54 | 2 | R 90 | R 180 | Mouser |
| 32 | Altera EPCS16 Serial Config Flash | EPCS16SI16N | SOIC-8 | 3 | R 90 | R 270 | Mouser |
| 33 | SPI NOR Flash 128Mbit 104 MHz | W25Q128JVSIQ | SOIC-8 | 4 | R 50 | R 200 | Communica |

**Subtotal: R 650**

---

## 6. Motor Control
*Projects 39 (FOC BLDC), 40 (PID Auto-Tune)*

| # | Component | Part Number | Package | Qty | Est. Unit | Est. Total | Supplier |
|---|---|---|---|---|---|---|---|
| 34 | 3-Phase BLDC Gate Driver + 2× CSA | DRV8302DCAR | HTSSOP-56 | 2 | R 210 | R 420 | RS Components ZA |
| 35 | N-Ch MOSFET 60V 30A TO-220 | IRLZ44NPBF | TO-220 | 12 | R 30 | R 360 | Communica |
| 36 | BLDC Gimbal Motor 24V 3-Phase | Generic 2804 | — | 1 | R 280 | R 280 | AliExpress |
| 37 | 14-bit Magnetic Encoder SPI | AS5047PAOS | TSSOP-14 | 2 | R 190 | R 380 | Mouser |
| 38 | Bootstrap Electrolytic 100µF 50V | Generic | TH | 6 | R 8 | R 48 | Communica |

**Subtotal: R 1 488**

---

## 7. FPGA Development & Test Hardware
*All 50 projects*

| # | Item | Specification | Qty | Est. Unit | Est. Total | Supplier |
|---|---|---|---|---|---|---|
| 39 | Cyclone III Dev Board EP3C16F484 | EP3C16F484C6N, 50 MHz osc, GPIO headers | 2 | R 900 | R 1 800 | AliExpress |
| 40 | USB Blaster II Clone (JTAG) | Quartus compatible, AS + JTAG modes | 1 | R 280 | R 280 | AliExpress |
| 41 | Logic Analyser 24 MHz 8-channel | Saleae clone, PulseView/Sigrok | 1 | R 350 | R 350 | AliExpress |
| 42 | Bench PSU 0–30V 0–5A | Dual-output, CC/CV, digital display | 1 | R 900 | R 900 | Mantech |
| 43 | AD9833 DDS Signal Generator Module | 0–12.5 MHz, SPI controlled from Pi 5 | 2 | R 60 | R 120 | AliExpress |
| 44 | SMD Breakout Boards SOIC/TSSOP | Various footprints for bench prototyping | 20 | R 10 | R 200 | AliExpress |
| 45 | Breadboard 830-point | Full size + jumper wire kit | 3 | R 60 | R 180 | Communica |
| 46 | SOIC-8 Test Clip | In-circuit SPI flash programming | 1 | R 120 | R 120 | AliExpress |

**Subtotal: R 3 950**

---

## 8. Passive Component Kits
*All analog, power, and filter projects*

| # | Kit | Specification | Qty | Est. Total | Supplier |
|---|---|---|---|---|---|
| 47 | SMD Resistor Kit 0402 1% | 1Ω – 1MΩ, 170 values × 50 pcs | 1 kit | R 180 | AliExpress |
| 48 | SMD Capacitor Kit 0402/0805 | 1pF – 100µF, 125 values × 50 pcs | 1 kit | R 220 | AliExpress |
| 49 | Electrolytic Capacitor Kit | 100µF–1000µF, 35V/50V rated | 20 pcs | R 120 | Communica |
| 50 | Precision Resistor Kit 0.1% 0603 | 10Ω–100kΩ, common E96 values | 1 kit | R 150 | RS Components ZA |
| 51 | Ferrite Bead Kit (EMI suppression) | 100Ω–1kΩ @ 100 MHz, 0603 | 1 kit | R 80 | AliExpress |
| 52 | Ceramic Cap 100nF 0402 decoupling | X7R 10V, bulk decoupling on all ICs | 200 pcs | R 60 | AliExpress |
| 53 | Crystal 50 MHz (FPGA board spare) | HC-49S, 20pF load | 3 pcs | R 45 | Communica |

**Subtotal: R 855**

---

## Cost Summary

| Domain | Projects | Subtotal |
|---|---|---|
| Analog Front-End & Signal Conditioning | 16–19 | R 1 480 |
| High-Speed ADC & DAC | 7, 15, 17, 27–30 | R 2 230 |
| Mil-Spec Power Supply & Protection | 13–15 | R 1 520 |
| Communication Interface ICs | 8–10, 31–34 | R 3 880 |
| Memory & Storage | 6, 30, 36, 42 | R 650 |
| Motor Control | 39–40 | R 1 488 |
| FPGA Development & Test Hardware | All | R 3 950 |
| Passive Component Kits | All | R 855 |
| **TOTAL** | | **≈ R 16 053** |

> Rounded project budget: **R 14 800 – R 16 500** depending on exchange rate and shipping.

---

## Procurement Notes

**RS Components ZA** — preferred for mil-spec and precision ICs (INA226, OPA657, IR2110, TVS, inductors). Reliable stock, datasheets guaranteed genuine.

**Mouser Electronics** — best for specialty parts: AD9226, AD5762, AS5047P, EPCS16. Use group orders to minimise shipping.

**Communica (JHB)** — walk-in or online, good for passives, connectors, MOSFETs, through-hole parts. Fastest for urgent stock.

**Mantech** — bench PSU, test equipment, lab consumables.

**AliExpress** — dev boards, clones, breakout boards, passive kits, non-critical discretes. Allow 2–4 weeks shipping. Order extras (10–20% buffer) for SOT-23/LFCSP ICs prone to loss during hand soldering.

---

## Critical Notes Per Domain

### MIL-STD-1553B (Project 33)
The UT63M125V is a genuine mil-spec part — verify stock with RS or contact DDC/Alta Data for alternatives. Budget ≈ R 700/unit. Holt HI-1573 is a lower-cost alternative at ≈ R 300 if budget is constrained.

### ARINC 429 (Project 34)
The HI-3593PQI from Holt Integrated Circuits is the standard choice. Confirm 3.3V I/O compatibility with Cyclone III LVTTL — level shifting may be needed.

### AD9226 ADC (Project 7)
Parallel LVTTL interface at 65 MSPS requires careful PCB layout — short traces, star ground, analogue/digital ground split. Do not prototype on breadboard; use a dedicated 4-layer PCB for this project.

### Cyclone III Dev Board (Item 39)
Verify the AliExpress board includes: EP3C16**F484** (not EP3C16E144), onboard EPCS16 config flash, 50 MHz oscillator, and accessible GPIO header pins. Some boards ship with EP3C25 — confirm before ordering.

### Power Supply Testing (Projects 13–15)
A 28V bench supply is mandatory for Mil-spec power projects. Do not use USB power. Start testing at 5V and ramp up with current limiting set to 500mA until circuit is verified.

---

## PCB Fabrication (Additional Budget)

For projects requiring dedicated PCBs (ADC bridge, power supply, motor drive):

| PCB | Layers | Est. Cost (JLCPCB 5 pcs) |
|---|---|---|
| ADC front-end board | 4-layer | ≈ R 280 |
| Mil-spec power supply | 2-layer | ≈ R 150 |
| Motor drive H-bridge | 2-layer | ≈ R 150 |
| General FPGA breakout | 2-layer | ≈ R 120 |

**PCB subtotal: ≈ R 700** (JLCPCB + DHL shipping to ZA)

---

*Document: `Cyclone_III_50_Projects_BOM.md`*  
*Repository: `Hardware-Design-Engineer-VHDL-Portfolio`*  
*Platform: Cyclone III EP3C16F484C6N + ESP32 + Raspberry Pi 3B/5*
