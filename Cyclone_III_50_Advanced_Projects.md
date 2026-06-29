# 50 Advanced FPGA Projects — Cyclone III EP3C16F484C6N
### Target Role: Hardware Design Engineer – Digital/Analog/VHDL | Saab Grintek Defence

**Hardware Platform:**
- **FPGA:** Intel/Altera Cyclone III EP3C16F484C6N (16K LEs, 56× M9K BRAM, 4× PLLs, 346 I/O)
- **SoC Co-processors:** ESP32 (Wi-Fi/BT, dual-core Xtensa), Raspberry Pi 3B / Pi 5 (ARM64 Linux)
- **Toolchain:** Quartus Prime Lite, ModelSim-Altera, LTSpice, KiCad/CircuitStudio

---

## DOMAIN 1 — VHDL HDL LIBRARY & REUSABLE BUILDING BLOCKS
*Maps to: "Develop common and reusable HDL library building blocks", VHDL competency*

---

### Project 1 — Generic FIFO Library IP Core

**Objective:** Build a parameterised, synthesisable FIFO library component usable across all subsequent projects.

**VHDL Architecture:**
```vhdl
-- Generic synchronous FIFO with configurable depth and width
entity generic_fifo is
  generic (
    DATA_WIDTH : integer := 8;
    ADDR_WIDTH : integer := 4   -- depth = 2^ADDR_WIDTH
  );
  port (
    clk      : in  std_logic;
    rst_n    : in  std_logic;
    wr_en    : in  std_logic;
    rd_en    : in  std_logic;
    din      : in  std_logic_vector(DATA_WIDTH-1 downto 0);
    dout     : out std_logic_vector(DATA_WIDTH-1 downto 0);
    full     : out std_logic;
    empty    : out std_logic;
    count    : out std_logic_vector(ADDR_WIDTH downto 0)
  );
end entity generic_fifo;

architecture rtl of generic_fifo is
  type mem_t is array (0 to 2**ADDR_WIDTH-1) of
    std_logic_vector(DATA_WIDTH-1 downto 0);
  signal mem             : mem_t;
  signal wr_ptr, rd_ptr  : unsigned(ADDR_WIDTH downto 0) := (others=>'0');
begin
  -- Write port
  process(clk)
  begin
    if rising_edge(clk) then
      if wr_en = '1' and full = '0' then
        mem(to_integer(wr_ptr(ADDR_WIDTH-1 downto 0))) <= din;
        wr_ptr <= wr_ptr + 1;
      end if;
    end if;
  end process;

  -- Read port
  process(clk)
  begin
    if rising_edge(clk) then
      if rd_en = '1' and empty = '0' then
        dout   <= mem(to_integer(rd_ptr(ADDR_WIDTH-1 downto 0)));
        rd_ptr <= rd_ptr + 1;
      end if;
    end if;
  end process;

  count <= std_logic_vector(wr_ptr - rd_ptr);
  full  <= '1' when (wr_ptr - rd_ptr) = 2**ADDR_WIDTH else '0';
  empty <= '1' when wr_ptr = rd_ptr else '0';
end architecture rtl;
```

**Variants to implement:**
- Async FIFO with Gray-code pointer crossing (CDC safe)
- Show-ahead (look-ahead) FIFO
- Dual-clock FIFO with BRAM inference targeting Cyclone III M9K blocks

**ModelSim Testbench Requirements:**
- Randomised write/read interleaving at 80% bus utilisation
- Full/empty corner case coverage
- Functional coverage group for pointer overflow

**Deliverables:** `lib/fifo/`, constraints file, timing report, functional coverage report

---

### Project 2 — CRC Engine Library (CRC-8/16/32)

**Objective:** Parameterised CRC computation core, single-cycle and pipelined variants.

**Key VHDL Concepts:**
- Generic polynomial constant
- Look-up table vs LFSR implementation
- Byte-serial and parallel (32-bit wide) architectures

**VHDL Skeleton:**
```vhdl
entity crc_engine is
  generic (
    POLY_WIDTH : integer := 32;
    POLYNOMIAL : std_logic_vector(31 downto 0) := x"04C11DB7"; -- CRC-32
    INIT_VALUE : std_logic_vector(31 downto 0) := x"FFFFFFFF";
    DATA_WIDTH : integer := 8
  );
  port (
    clk    : in  std_logic;
    rst_n  : in  std_logic;
    valid  : in  std_logic;
    data   : in  std_logic_vector(DATA_WIDTH-1 downto 0);
    crc    : out std_logic_vector(POLY_WIDTH-1 downto 0);
    done   : out std_logic
  );
end entity crc_engine;
```

**Integration:** Connect to UART/SPI receivers in later projects. Validate against Python `crcmod` reference.

**Resource Target:** < 150 LEs, 1 M9K for LUT variant on EP3C16.

---

### Project 3 — Parameterised UART Transceiver Library

**Objective:** Full-duplex UART with configurable baud, parity, and stop bits. Foundation for all serial comms projects.

**Features:**
- Baud rate synthesised from PLL output (50 MHz system clock → any standard baud)
- 16× oversampling for robust RX
- FIFO-backed TX/RX (instantiate Project 1)
- Framing error, parity error, overrun error flags

**VHDL Top-level ports:**
```vhdl
entity uart_top is
  generic (
    CLK_FREQ  : integer := 50_000_000;
    BAUD_RATE : integer := 115200;
    DATA_BITS : integer := 8;        -- 5-9
    PARITY    : string  := "NONE";   -- "NONE","ODD","EVEN"
    STOP_BITS : integer := 1         -- 1 or 2
  );
  port (
    clk      : in  std_logic;
    rst_n    : in  std_logic;
    -- TX
    tx_data  : in  std_logic_vector(DATA_BITS-1 downto 0);
    tx_valid : in  std_logic;
    tx_ready : out std_logic;
    tx_pin   : out std_logic;
    -- RX
    rx_pin   : in  std_logic;
    rx_data  : out std_logic_vector(DATA_BITS-1 downto 0);
    rx_valid : out std_logic;
    rx_error : out std_logic_vector(2 downto 0)  -- [overrun,frame,parity]
  );
end entity uart_top;
```

**Testing:** Loopback test on FPGA, cross-check with ESP32 UART peripheral at 921600 baud.

---

### Project 4 — SPI Master/Slave Library

**Objective:** Quad-mode SPI library with CPOL/CPHA configuration and DMA-style burst transfer.

**Modes implemented:** CPOL=0/CPHA=0, CPOL=0/CPHA=1, CPOL=1/CPHA=0, CPOL=1/CPHA=1

**Key VHDL:**
- Shift register as generic width (`DATA_WIDTH` generic)
- CS management with configurable setup/hold times in clock cycles
- Interrupt output on transaction complete

**Hardware targets:** Drive ADC (AD9226 or similar) and DAC (DAC8811) for later analog interfacing projects.

---

### Project 5 — I²C Master Controller Library

**Objective:** Multi-master I²C with clock stretching, 7/10-bit addressing, repeated-start.

**State machine states:** `IDLE → START → ADDR → ACK_ADDR → DATA → ACK_DATA → STOP`

**VHDL FSM template:**
```vhdl
type i2c_state_t is (
  ST_IDLE, ST_START, ST_ADDR, ST_ACK_ADDR,
  ST_WR_DATA, ST_RD_DATA, ST_ACK_DATA,
  ST_NACK, ST_STOP, ST_ERROR
);
signal state : i2c_state_t := ST_IDLE;
```

**Integration:** Pi 5 acts as I²C slave (via bitbang or SCB peripheral); FPGA is master polling a BME280 pressure sensor routed through it.

---

## DOMAIN 2 — HIGH-SPEED DIGITAL HARDWARE & INTERFACES
*Maps to: "Design of high-speed digital hardware using FPGA SOCs", CAN/PCIe/GigE/Aurora/DDR*

---

### Project 6 — DDR-Style SDRAM Controller (SDRAM via Cyclone III)

**Objective:** Build a full SDRAM controller targeting an IS42S16400J SDRAM (or similar) — initialisation sequence, auto-refresh, bank management, burst access.

**Architecture:**
```
[ Wishbone Bus Interface ]
        |
[ Command Scheduler ] — [ Refresh Timer ]
        |
[ Row/Column Address Mux ]
        |
[ SDRAM PHY Interface (registered I/O) ]
        |
[ IS42S16400J 64Mbit SDRAM ]
```

**VHDL highlights:**
- CAS latency pipeline alignment
- Distributed refresh with tREF tracking
- Address mapping: {bank[1:0], row[12:0], col[8:0]}

**Performance target:** ≥ 80 MB/s sustained read throughput at 100 MHz.

---

### Project 7 — High-Speed SPI-to-Parallel ADC Bridge (12-bit, 50 MSPS simulation)

**Objective:** Interface a high-speed ADC (AD9226 or AD9203) to Cyclone III LVTTL I/O and stream samples into FIFO for Pi 5 DMA ingestion.

**Pipeline:**
```
ADC DATA[11:0] → [ Input Register (DDR capture) ] → [ Demux 1:4 ] 
    → [ Decimation Filter ] → [ FIFO ] → [ SPI Slave to Pi 5 ]
```

**VHDL techniques:**
- `ALTDDIO_IN` megafunction for double-data-rate capture
- Pipelined two's complement correction
- Overflow/underflow clipping

**Validation:** Inject a 1 kHz sine from signal generator; FFT on Pi 5 to measure SFDR.

---

### Project 8 — CAN Bus Controller (ISO 11898)

**Objective:** Implement CAN 2.0B protocol controller in VHDL — bit stuffing, CRC-15, ACK, error frames, arbitration.

**Block diagram:**
```
[ CAN_RX pin ] → [ Bit Sampler (3× oversampling) ] → [ Bit Unstuffer ]
                                                          → [ Frame Parser ]
                                                          → [ CRC Checker ]
                                                          → [ ACK Generator ]
[ Frame Builder ] → [ Bit Stuffer ] → [ CAN_TX pin ]
```

**Key parameters:**
- CAN clock: derived from PLL (16 MHz nominal, configurable TQ)
- Sync-seg: 1 TQ, Prop+Phase: configurable via registers
- Error counter: TEC/REC per ISO 11898-1 §6.12

**Integration:** ESP32 as CAN node (TWAI peripheral) with MCP2551 transceiver; FPGA controller exchanges CANopen SDO messages.

---

### Project 9 — Gigabit Ethernet MAC (GMII Interface)

**Objective:** Implement IEEE 802.3 Ethernet MAC layer — preamble, SFD, FCS (CRC-32), IFG, collision detection stub.

**VHDL Architecture:**
```vhdl
-- TX path
entity eth_mac_tx is
  port (
    clk_125    : in  std_logic;  -- 125 MHz from PLL
    rst_n      : in  std_logic;
    -- User interface (AXI-Stream-like)
    s_axis_data  : in  std_logic_vector(7 downto 0);
    s_axis_valid : in  std_logic;
    s_axis_last  : in  std_logic;
    s_axis_ready : out std_logic;
    -- GMII
    gmii_txd   : out std_logic_vector(7 downto 0);
    gmii_tx_en : out std_logic;
    gmii_tx_er : out std_logic
  );
end entity eth_mac_tx;
```

**Integration:** Connect to KSZ9031/RTL8211 PHY on RMII (10/100) since Cyclone III lacks built-in SERDES for true GigE — use MII at 100 Mbps with the MAC logic fully validated; document limitation and migration path to Cyclone V/Arria.

**Pi 5 role:** Raw socket receiver validating frame integrity.

---

### Project 10 — Aurora 8B/10B Serial Link (Emulated LVDS)

**Objective:** Implement a simplified Aurora-like 8B/10B encoded serial link over LVDS pairs using Cyclone III LVDS I/O.

**8B/10B encoder in VHDL:**
```vhdl
entity enc_8b10b is
  port (
    clk      : in  std_logic;
    rst_n    : in  std_logic;
    data_in  : in  std_logic_vector(7 downto 0);
    k_char   : in  std_logic;   -- control character flag
    data_out : out std_logic_vector(9 downto 0);
    disp_out : out std_logic    -- running disparity
  );
end entity enc_8b10b;
```

**Link framing:** K28.5 comma character for word alignment, K27.7/K29.7 for flow control.

**Physical:** FPGA LVDS output pair → cable → FPGA LVDS input pair (loopback) or → Pi 5 SERDES via SN65LVDS31 driver.

**BER test:** PRBS-23 pattern; measure bit errors over 10⁹ bits.

---

### Project 11 — PCIe-over-Parallel Bus Bridge

**Objective:** Since EP3C16 lacks native PCIe SERDES, implement a TLP (Transaction Layer Packet) framing engine over a 32-bit parallel bus — the logic that would feed a PCIe PHY in a larger device.

**Scope:** Memory read/write TLP encoder/decoder, completion generation, tag management.

**VHDL modules:**
- `tlp_encoder.vhd` — builds MRd/MWr/CplD TLPs from user transactions
- `tlp_decoder.vhd` — parses incoming byte stream into structured transactions
- `tag_manager.vhd` — tracks outstanding non-posted requests (4-bit tag pool)

**Testbench:** Pi 5 software generates expected TLP byte sequences; compare against VHDL output.

---

### Project 12 — High-Speed DDR LVDS Serdes (Source-Synchronous Capture)

**Objective:** Implement source-synchronous 4-lane LVDS data capture with forwarded clock — typical of camera sensors and RF ADCs.

**Cyclone III primitives used:**
- `ALTLVDS_RX` megafunction — deserialises 7:1 ratio
- `ALTPLL` — generates 7× bit clock from input word clock
- `ALTDDIO_IN` — DDR flip-flops at I/O pads

**Output:** 32-bit parallel word at word-clock rate → FIFO → Pi 5 via SPI DMA

---

## DOMAIN 3 — MIL SPEC POWER SUPPLY DESIGN
*Maps to: "Mil Spec Power supply design", LTSpice simulation, analog competency*

---

### Project 13 — Mil Spec DC-DC Buck Converter (28V → 5V, 10A) with FPGA Supervisory Control

**Objective:** Design a synchronous buck converter meeting MIL-STD-704F (28 VDC aircraft bus), with FPGA performing digital PWM control, current limiting, sequencing, and telemetry.

**LTSpice Simulation Schematic (key components):**
```
VIN=28V → [ EMI Filter (MIL-STD-461 compliant) ] → [ Gate Driver IR2110 ]
         → [ Q1 Hi-Side IRF3710 ] → [ L1 10µH/15A ] → VOUT=5V
                                  → [ Q2 Lo-Side IRF3710 ] → GND
VOUT → [ Current Sense (0.01Ω, INA226) ] → FPGA ADC
VOUT → [ Voltage Divider ] → FPGA ADC
FPGA PWM → Gate Driver bootstrap circuit
```

**FPGA Digital Control Loop (VHDL):**
```vhdl
entity dpwm_controller is
  generic (
    PWM_RES  : integer := 10;   -- 10-bit resolution
    CLK_FREQ : integer := 50_000_000;
    SW_FREQ  : integer := 200_000  -- 200 kHz switching
  );
  port (
    clk      : in  std_logic;
    rst_n    : in  std_logic;
    v_sense  : in  std_logic_vector(11 downto 0);  -- from ADC
    i_sense  : in  std_logic_vector(11 downto 0);
    v_ref    : in  std_logic_vector(11 downto 0);  -- setpoint
    pwm_out  : out std_logic;
    fault    : out std_logic
  );
end entity dpwm_controller;
```

**PI Compensator:** Implemented as fixed-point arithmetic (Q12.4 format)

**Protection features:**
- OCP: cycle-by-cycle current limiting at 110% rated
- OVP: crowbar trigger within 1 µs via direct FPGA GPIO
- UVL: inhibit below 18V input
- Thermal: NTC thermistor ADC, derate above 85°C

**MIL-STD-461 EMI Filter:** Two-stage LC with CM choke, X/Y capacitors; verified in LTSpice.

**Deliverables:** LTSpice simulation files, VHDL control loop, Bode plot of compensated loop (crossover >20 kHz, phase margin >45°), load transient waveform captures.

---

### Project 14 — Mil Spec Power Sequencer & Housekeeping Controller

**Objective:** FPGA-based power rail sequencing for a multi-voltage system (3.3V, 5V, 12V, 28V), with PGOOD monitoring, fault logging, and UART telemetry to Pi 5.

**Sequencing state machine:**
```
STARTUP:
  t=0ms   : Assert EN_28V, wait PGOOD_28V
  t=10ms  : Assert EN_12V, wait PGOOD_12V
  t=20ms  : Assert EN_5V,  wait PGOOD_5V
  t=30ms  : Assert EN_3V3, wait PGOOD_3V3
  t=40ms  : SYSTEM_READY

SHUTDOWN (reverse order with 5ms gaps)
FAULT: any PGOOD lost → immediate shutdown all rails, log fault code
```

**VHDL state machine:** Encodes 4-bit rail status, logs fault timestamp using 32-bit free-running counter.

**Telemetry frame (UART to Pi 5):**
- 16-byte packet: `[SOF][RAIL_STATUS][V1][V2][V3][V4][I1][I2][FAULT_CODE][TIMESTAMP_32][CRC16][EOF]`

---

### Project 15 — Digital Programmable Voltage Reference (16-bit DAC + FPGA)

**Objective:** FPGA generates 16-bit SPI commands to an AD5762 precision DAC, implementing a digitally-programmable Mil-spec reference voltage with <1 LSB INL across -55°C to +125°C.

**Calibration procedure (VHDL):**
- Store factory calibration coefficients in on-chip M9K as ROM
- Apply gain/offset correction: `corrected = raw * gain_coeff + offset_coeff`
- All arithmetic in Q16.16 fixed-point

**Temperature compensation:** Read PT100 RTD via 16-bit ADC (ADS1115 over I²C); look-up compensation table from M9K ROM.

---

## DOMAIN 4 — LOW-SIGNAL AMPLIFICATION & ANALOG FRONT-END
*Maps to: "Low Signal Amplification", "Photo diode detector / Laser diode driving circuits"*

---

### Project 16 — Transimpedance Amplifier (TIA) Front-End for PIN Photodiode

**Objective:** Design LTSpice simulation and PCB schematic for a PIN photodiode TIA, with FPGA controlling gain-switching relays and digitising the output.

**LTSpice TIA Design:**
```
                    Rf = 100kΩ (gain-switched: 10k/100k/1M)
                    Cf = 1pF  (stability)
PD_cathode ────────────────────────────┐
                                        ├── OPA657 (1.6 GHz GBW)
PD_anode ── Vbias ─────────────────────┘── VOUT
                                              |
                                        [ Anti-alias LPF ]
                                              |
                                        [ FPGA ADC input ]
```

**FPGA control:**
- 3 GPIO lines select Rf via CMOS switches (ADG733)
- Auto-ranging: measure output, if >90% FSR → switch to lower gain range
- Log current range + ADC reading in timestamped FIFO

**Noise budget (LTSpice AC analysis):**
- Input referred noise: < 2 pA/√Hz at 1 kHz
- Dynamic range: > 80 dB

**Deliverables:** LTSpice .asc, noise simulation, VHDL auto-ranging FSM, SNR measurement log.

---

### Project 17 — Laser Diode Driver with APC (Automatic Power Control)

**Objective:** FPGA-controlled CW/pulsed laser diode driver with monitor photodiode feedback loop.

**Circuit (LTSpice):**
```
FPGA DAC (12-bit SPI) → [ V-to-I Converter (OPA2134) ] → [ LD Cathode ]
                                                          → [ LD Anode → GND ]
Monitor PD → [ TIA (OPA657) ] → [ ADC ] → FPGA feedback
```

**FPGA APC control loop:**
- Target optical power set via register write from Pi 5 over UART
- PI loop at 10 kHz update rate
- Soft-start: ramp current from 0 to I_th over 10 ms to prevent facet damage
- OCP: hardware comparator trips gate in <100 ns, FPGA sets fault flag

**Pulsed mode:**
- Arbitrary pulse pattern loaded into 1024-deep BRAM pattern generator
- Pulse width: 10 ns minimum (single FPGA clock at 100 MHz)

---

### Project 18 — Lock-In Amplifier (Digital, FPGA-based)

**Objective:** Digital lock-in amplifier for extracting signals buried in noise, using FPGA multiplier blocks and CIC decimation filters.

**Signal flow:**
```
Signal input → ADC (12-bit, 1 MSPS) → [ Multiplier × cos(ωt) ] → [ CIC Filter ]
Reference gen (FPGA DDS)           → [ Multiplier × sin(ωt) ] → [ CIC Filter ]
                                                                         ↓
                                                               [ R = √(I²+Q²) ]
                                                               [ θ = atan2(Q,I) ]
```

**VHDL DDS (Direct Digital Synthesis):**
```vhdl
entity dds is
  generic (
    PHASE_WIDTH : integer := 32;
    AMP_WIDTH   : integer := 16
  );
  port (
    clk       : in  std_logic;
    rst_n     : in  std_logic;
    freq_word : in  std_logic_vector(PHASE_WIDTH-1 downto 0);
    sine_out  : out std_logic_vector(AMP_WIDTH-1 downto 0);
    cosine_out: out std_logic_vector(AMP_WIDTH-1 downto 0)
  );
end entity dds;
-- Phase accumulator → quarter-wave LUT in M9K → full-wave reconstruction
```

**Application:** Detect modulated signal at 1 kHz with SNR improvement from -20 dB to >30 dB.

---

### Project 19 — Wideband Variable-Gain Amplifier (VGA) Controller

**Objective:** FPGA controls AD8338 VGA gain over SPI for an automatic gain control (AGC) loop, targeting ±0.5 dB gain flatness from 1 MHz to 18 MHz.

**AGC loop:** RMS detector implemented in VHDL (squarer → IIR averager → sqrt Newton-Raphson) feeds PI controller driving VGA gain word.

**Calibration:** Factory gain-vs-code table stored in M9K ROM, applied as gain linearisation.

---

## DOMAIN 5 — TEST ENVIRONMENTS & VERIFICATION
*Maps to: "Design of test environments using tools such as ModelSim", test documentation*

---

### Project 20 — Universal Verification Testbench Framework (VHDL-2008)

**Objective:** Build a reusable testbench infrastructure in VHDL-2008 with constrained-random stimulus, functional coverage, and self-checking.

**Framework components:**
```vhdl
-- Scoreboard package
package scoreboard_pkg is
  type scoreboard_t is protected
    procedure push(data : std_logic_vector);
    impure function pop return std_logic_vector;
    impure function is_empty return boolean;
    impure function errors return integer;
  end protected scoreboard_t;
end package;

-- Coverage group example
procedure sample_coverage(din : std_logic_vector(7 downto 0)) is
begin
  -- Track all 256 byte values seen
  cov_array(to_integer(unsigned(din))) := '1';
end procedure;

impure function get_coverage return real is
  variable cnt : integer := 0;
begin
  for i in cov_array'range loop
    if cov_array(i) = '1' then cnt := cnt + 1; end if;
  end loop;
  return real(cnt) / real(cov_array'length) * 100.0;
end function;
```

**Agents implemented:**
- UART BFM (Bus Functional Model)
- SPI master/slave BFM
- AXI-Stream source/sink
- Memory model (for SDRAM controller verification)

---

### Project 21 — Fault Injection Test Environment

**Objective:** ModelSim-based fault injection framework for verifying error detection and recovery in Projects 8 (CAN), 9 (Ethernet), 19 (VGA AGC).

**Fault types modelled:**
- Bit flip (single/double) via XOR injection
- Stuck-at-0 / stuck-at-1 on bus lines
- Timing glitch (hold time violation) using VHDL transport delay
- Power brownout (voltage dip during write) — modelled as synchronisation loss

**VHDL fault injector:**
```vhdl
entity fault_injector is
  generic (
    FAULT_PROB  : integer := 100;  -- 1 fault per 100 transactions
    FAULT_TYPE  : string  := "BIT_FLIP"
  );
  port (
    clk        : in  std_logic;
    data_in    : in  std_logic_vector(7 downto 0);
    data_out   : out std_logic_vector(7 downto 0);
    fault_active : out std_logic
  );
end entity fault_injector;
```

---

### Project 22 — Protocol Compliance Checker (Assertions)

**Objective:** Write a comprehensive PSL/VHDL assertion checker for SPI, I²C, and UART protocols usable as a passive monitor in ModelSim.

**VHDL-2008 assertions:**
```vhdl
-- SPI CS must be low for full transaction
assert_spi_cs : assert always (spi_cs = '0' -> (spi_clk'event));

-- UART start bit must be followed by 8 data bits
assert_uart_frame : assert always (uart_rx = '0' -> 
  next[8] (uart_rx = stop_bit_value));
```

**ModelSim usage:** Compile checker as a separate library, bind to DUT port map — zero simulation overhead when disabled via generic.

---

### Project 23 — Hardware-in-the-Loop (HIL) Test Rig

**Objective:** FPGA as the HIL stimulator — generates deterministic hardware stimulus (signals, power events, bus traffic) while Pi 5 runs test orchestration software.

**Architecture:**
```
Pi 5 (pytest + fabric) 
    │  UART command channel
    ▼
FPGA (stimulus generator + response checker)
    │  GPIO / SPI / UART / CAN
    ▼
DUT (Device Under Test — any project PCB)
```

**FPGA test sequences (VHDL):**
- Stored in M9K as binary test vectors
- Loaded via UART from Pi 5 at runtime
- Results timestamped with 1 µs resolution and returned to Pi 5

---

## DOMAIN 6 — EMI COMPLIANCE & SIGNAL INTEGRITY
*Maps to: "EMI compliance", "EMI design concepts", HyperLynx PCB analysis*

---

### Project 24 — Spread-Spectrum Clock Generator (FPGA PLL + VHDL Control)

**Objective:** Implement FPGA-controlled spread-spectrum clocking using the EP3C16 PLL fractional-N modulation to reduce EMI peaks by >10 dBµV/m.

**VHDL modulation:**
```vhdl
entity ssc_modulator is
  generic (
    MOD_DEPTH  : real := 0.005;  -- ±0.5% frequency deviation
    MOD_FREQ   : integer := 33000  -- 33 kHz modulation rate
  );
  port (
    clk_ref    : in  std_logic;
    mod_enable : in  std_logic;
    pll_m_word : out std_logic_vector(7 downto 0)  -- to PLL reconfig
  );
end entity ssc_modulator;
```

**PLL reconfiguration:** Use Cyclone III PLL dynamic reconfiguration (`ALTPLL_RECONFIG`) to dither M/N values.

**Measurement:** Near-field EMI scan before/after on a PCB with 50 MHz clock trace, log reduction in dBµV/m.

---

### Project 25 — Impedance-Controlled PCB Design & SI Simulation

**Objective:** Design a 4-layer PCB for the FPGA board with controlled impedance traces (50Ω single-ended, 100Ω differential), and simulate in HyperLynx.

**Stackup:**
```
Layer 1 (top):    Signal — 0.12mm Cu
Layer 2:          GND plane — 0.035mm Cu
                  [FR4 core — 0.2mm]
Layer 3:          Power plane — 0.035mm Cu
                  [FR4 core — 0.2mm]
Layer 4 (bot):    Signal — 0.12mm Cu
```

**HyperLynx simulations:**
- Eye diagram on 100 MHz LVDS pair (project 12 signals)
- Crosstalk analysis: aggressor LVDS → victim SPI line
- Power delivery network (PDN) impedance: target <10 mΩ at 100 MHz

**Deliverables:** HyperLynx .hyp file, eye diagram screenshot, PDN impedance curve, design rule notes.

---

### Project 26 — FPGA I/O Drive Strength & Slew Rate Optimiser

**Objective:** Characterise and optimise EP3C16 output drive strength (4/8/12/16 mA) and slew rate settings for minimum EMI while meeting timing.

**Automated sweep:**
- FPGA generates 50 MHz clock with each drive/slew combination
- Pi 5 via GPIO triggers a capture on a USB oscilloscope (Rigol DS1054Z over LAN)
- Measure rise time, overshoot, radiated emission (near-field probe if available)

**VHDL generation:**
```vhdl
-- Quartus attribute for drive strength
attribute CURRENT_STRENGTH_NEW : string;
attribute CURRENT_STRENGTH_NEW of clk_out_pin : signal is "8MA";
attribute SLEW_RATE : integer;
attribute SLEW_RATE of clk_out_pin : signal is 1;  -- 0=slow, 1=fast
```

**Output:** Heat-map table of EMI vs timing margin for each setting combination.

---

## DOMAIN 7 — DIGITAL SIGNAL PROCESSING & WAVEFORM PROCESSING
*Maps to: A2D/D2A high speed, SERDES, precision measurement*

---

### Project 27 — FIR Filter Bank (Polyphase, Reconfigurable Coefficients)

**Objective:** Implement a 64-tap FIR filter with runtime-loadable coefficients from Pi 5 via SPI, targeting 8× polyphase decimation.

**VHDL architecture:**
```vhdl
entity fir_filter is
  generic (
    TAPS       : integer := 64;
    DATA_WIDTH : integer := 16;
    COEF_WIDTH : integer := 18
  );
  port (
    clk       : in  std_logic;
    rst_n     : in  std_logic;
    -- Coefficient loading (SPI slave)
    spi_clk   : in  std_logic;
    spi_mosi  : in  std_logic;
    spi_cs_n  : in  std_logic;
    -- Data path
    din       : in  signed(DATA_WIDTH-1 downto 0);
    din_valid : in  std_logic;
    dout      : out signed(DATA_WIDTH+COEF_WIDTH-1 downto 0);
    dout_valid: out std_logic
  );
end entity fir_filter;
```

**DSP mapping:** Each multiply-accumulate maps to Cyclone III embedded 18×18 multiplier blocks (EP3C16 has 56 multipliers).

**Validation:** 1 kHz passband, 2 kHz stopband; measure stopband attenuation >60 dB using Pi 5 FFT.

---

### Project 28 — Real-Time FFT Spectrum Analyser (64-point, Radix-2 Cooley-Tukey)

**Objective:** Streaming 64-point FFT in VHDL, outputting magnitude spectrum in real time to Pi 5 for display.

**Architecture:** Radix-2 DIT FFT butterfly, 6 stages for 64-point, twiddle factors in M9K ROM.

**VHDL butterfly unit:**
```vhdl
entity butterfly is
  generic (DATA_W : integer := 16; TWIDDLE_W : integer := 16);
  port (
    clk      : in  std_logic;
    a_re, a_im : in  signed(DATA_W-1 downto 0);
    b_re, b_im : in  signed(DATA_W-1 downto 0);
    w_re, w_im : in  signed(TWIDDLE_W-1 downto 0);  -- twiddle
    y0_re, y0_im : out signed(DATA_W downto 0);
    y1_re, y1_im : out signed(DATA_W downto 0)
  );
end entity butterfly;
```

**Pi 5 display:** Python `matplotlib` real-time waterfall spectrogram over UART at 921600 baud.

---

### Project 29 — Digital Frequency Meter & Phase Comparator

**Objective:** Measure frequency (1 Hz to 50 MHz) and phase difference between two signals with 1 ppm accuracy using FPGA timer/counter.

**Techniques:**
- Reciprocal counting for low frequencies (<1 kHz): measure period, compute F = 1/T
- Direct counting for high frequencies: count edges in 1-second gate
- Phase: TDC (Time-to-Digital Converter) using carry-chain delay line

**VHDL TDC:**
```vhdl
-- Carry chain TDC — maps to Cyclone III fast carry chains
-- Resolution ≈ 200 ps (1 carry cell delay)
entity tdc is
  generic (STAGES : integer := 64);
  port (
    start  : in  std_logic;
    stop   : in  std_logic;
    result : out std_logic_vector(STAGES-1 downto 0)  -- thermometer code
  );
end entity tdc;
```

**Calibration:** Measure internal PLL output frequency against GPS 1PPS reference (via Pi 5 GPIO).

---

### Project 30 — Waveform Capture & Digital Storage Oscilloscope (DSO) Core

**Objective:** FPGA-based DSO front-end capturing 12-bit samples at 50 MSPS with hardware triggering, storing waveforms in SDRAM (Project 6) for Pi 5 readout.

**Trigger modes (VHDL FSM):**
- Edge trigger: rising/falling with hysteresis
- Pulse-width trigger: capture pulses narrower or wider than threshold
- Runt trigger: pulse that doesn't cross both thresholds
- Pattern trigger: 8-bit parallel bus pattern match

**Pre-trigger memory:** Circular SDRAM buffer, configurable 0-100% pre-trigger depth.

**Pi 5 readout:** Waveform data over SPI DMA; Pi renders via `matplotlib` — recreates basic oscilloscope UI.

---

## DOMAIN 8 — COMMUNICATION PROTOCOL BRIDGING & GATEWAYS
*Leverages existing MPT/SIPE experience, maps to Saab system integration needs*

---

### Project 31 — Multi-Protocol Serial Concentrator (UART/SPI/I²C → Parallel Bus)

**Objective:** FPGA acts as a protocol concentrator — receives data from 4× UART, 2× SPI, 2× I²C simultaneously and presents a unified 32-bit parallel memory-mapped interface to Pi 5.

**Memory map (FPGA register bank):**
```
0x000 — 0x0FF : UART0 RX FIFO (256 bytes)
0x100 — 0x1FF : UART1 RX FIFO
0x200 — 0x2FF : SPI0 RX FIFO
0x300 — 0x3FF : I2C0 RX FIFO
0x400         : Status register (FIFO occupancy bitmap)
0x404         : Interrupt mask
0x408         : Interrupt status (write-1-to-clear)
```

**Pi 5 driver:** Linux kernel module (or UIO + userspace) accesses FPGA registers over GPIO bitbang parallel bus.

---

### Project 32 — HDLC/PPP Frame Engine

**Objective:** VHDL implementation of HDLC framing (flag bytes, bit stuffing, FCS) for point-to-point serial links used in avionics and defence comms.

**HDLC frame format:**
```
[ 0x7E ] [ Address ] [ Control ] [ Information ] [ FCS-16 ] [ 0x7E ]
```

**VHDL modules:**
- `hdlc_tx.vhd` — bit-stuffer, FCS-16 appender, flag inserter
- `hdlc_rx.vhd` — flag detector, bit-destuffer, FCS checker, frame boundary detector
- `hdlc_top.vhd` — integrates TX/RX with AXI-Stream interfaces

**Test:** Transfer 10,000 frames of randomised payload; verify zero FCS errors and 100% payload integrity.

---

### Project 33 — MIL-STD-1553B Bus Controller Stub

**Objective:** Implement a MIL-STD-1553B Bus Controller (BC) in VHDL — command word generation, Manchester-II encoding, RT response validation.

**Manchester-II encoder:**
```vhdl
entity manchester_enc is
  port (
    clk    : in  std_logic;  -- 2× bit rate (2 MHz for 1553)
    data   : in  std_logic;
    sync   : in  std_logic;  -- command or data sync pulse
    output : out std_logic
  );
end entity manchester_enc;
-- '1' → 1→0 transition mid-bit
-- '0' → 0→1 transition mid-bit
```

**Command word structure (16-bit):**
- [15:11] RT Address, [10] T/R, [9:5] Subaddress, [4:0] Word count

**Scope:** BC-only (no RT or BM) — FPGA polls 4 simulated RT nodes and logs responses.

---

### Project 34 — ARINC 429 Transceiver

**Objective:** Implement ARINC 429 bipolar RZ transmitter and receiver in VHDL, usable for avionics data bus interfacing.

**Signal encoding:** Bipolar Return-to-Zero (HI=+10V, NULL=0, LO=-10V)

**FPGA implementation:**
- TX: FPGA differential pair → gate driver (HI/LO select) → transformer coupling → ARINC 429 bus
- RX: transformer → comparator → FPGA differential input → Manchester decoder

**Word format (32-bit):**
```
[31:30] Parity + SSM | [29:11] Data | [10:8] SDI | [7:0] Label (octal)
```

**Parity:** Odd parity across all 32 bits (bit 32 set to make total '1' count odd).

---

## DOMAIN 9 — FPGA SOC INTEGRATION WITH LINUX (PI 5 ARM64)
*Maps to: "FPGA SOC", "Interface with project management", documentation*

---

### Project 35 — FPGA as Linux Peripheral (UIO + Device Tree)

**Objective:** Expose FPGA logic as a Linux UIO (Userspace I/O) device on Pi 5, with proper device tree overlay, interrupt handling, and POSIX read/write interface.

**Device Tree Overlay (`fpga-peripheral.dts`):**
```dts
/dts-v1/;
/plugin/;
/ {
  compatible = "brcm,bcm2712";
  fragment@0 {
    target = <&gpio>;
    __overlay__ {
      fpga_pins: fpga_pins {
        brcm,pins = <17 18 19 20 21 22 23 24>;
        brcm,function = <0>;  /* input */
        brcm,pull = <1>;      /* pull-down */
      };
    };
  };
  fragment@1 {
    target-path = "/";
    __overlay__ {
      fpga_dev: fpga-peripheral {
        compatible = "generic-uio";
        reg = <0x0 0xFE200000 0x0 0x1000>;
        interrupts = <0 49 4>;
        interrupt-parent = <&intc>;
        status = "okay";
      };
    };
  };
};
```

**Userspace driver (C):** `open("/dev/uio0")`, `mmap()` for register access, `read()` blocks on interrupt.

---

### Project 36 — Real-Time Data Logger (FPGA Timestamping + Pi 5 Storage)

**Objective:** FPGA timestamps events with 10 ns resolution (100 MHz counter) and streams to Pi 5 for TimescaleDB insertion.

**FPGA event capture:**
- 16 independent event channels (GPIO inputs)
- Each event: `{channel[3:0], timestamp[47:0], edge[0]}` = 7 bytes
- 256-deep event FIFO per channel
- DMA-style burst transfer to Pi 5 over high-speed SPI (20 MHz)

**Pi 5 ingestion daemon (Python asyncio):**
```python
async def ingest_events(spi_dev, db_pool):
    while True:
        raw = await spi_dev.read(4096)  # 4KB burst
        events = parse_event_frames(raw)
        async with db_pool.acquire() as conn:
            await conn.executemany(
                "INSERT INTO events(channel, ts_ns, edge) VALUES($1,$2,$3)",
                events
            )
```

---

### Project 37 — FPGA-Accelerated CRC for High-Speed Network Storage

**Objective:** Offload CRC-32C (iSCSI) computation to FPGA, processing 32-bit words at line rate, achieving >3 Gbps throughput.

**Parallel CRC algorithm:** Process 4 bytes/cycle using pre-computed table method.

**AXI-Stream interface:**
- `s_axis_data[31:0]`, `s_axis_keep[3:0]`, `s_axis_last`
- Output: `crc32_result[31:0]` valid one cycle after `s_axis_last`

**Benchmark:** Measure CRC throughput using Pi 5 SPI DMA vs software CRC on ARM; target >10× speedup.

---

### Project 38 — ESP32 ↔ FPGA ↔ Pi 5 IoT Gateway

**Objective:** Three-tier gateway: ESP32 collects sensor data over BLE/Wi-Fi, FPGA buffers and timestamp-stamps it, Pi 5 provides cloud uplink and visualisation.

**Protocol stack:**
```
Sensor (I²C/SPI) → ESP32 → [UART 921600] → FPGA → [SPI DMA] → Pi 5 → MQTT → Cloud
                              timestamping
                              aggregation
                              alarm detection
```

**FPGA alarm engine (VHDL):**
- Threshold comparators: 8 configurable channels, upper/lower bounds
- Hysteresis: configurable dead-band per channel
- Rate-of-change detector: differentiate samples over 100ms window

**ESP32 firmware (Arduino/IDF):** BLE GATT server + Wi-Fi provisioning; UART frame format matches FPGA parser.

---

## DOMAIN 10 — DIGITAL CONTROL SYSTEMS
*Maps to: test, integration, debug, system-level projects*

---

### Project 39 — BLDC Motor Controller (Field-Oriented Control, FPGA-based)

**Objective:** FPGA implements Clarke/Park transforms, PI current controllers, SVPWM generation — all in fixed-point VHDL — for a 3-phase BLDC motor.

**Control chain:**
```
Hall sensors → [ Position/Speed estimator ] → [ Park transform⁻¹ ]
ADC (phase currents) → [ Clarke transform ] → [ Park transform ]
                                    → [ Id PI controller ] → [ SVPWM ]
                                    → [ Iq PI controller ] → [ 6× PWM outputs ]
```

**Fixed-point arithmetic:** Q1.15 format throughout; saturation logic on all accumulators.

**VHDL Clarke transform:**
```vhdl
-- Iα = Ia, Iβ = (Ia + 2Ib)/√3
Ialpha <= ia;
Ibeta  <= resize(shift_right(ia + shift_left(ib,1), 0) * INV_SQRT3, DATA_W);
-- INV_SQRT3 = 0x49E7 in Q0.16
```

**PWM frequency:** 20 kHz; dead-time insertion: 500 ns (25 clock cycles at 50 MHz).

---

### Project 40 — PID Controller with Auto-Tune (Ziegler-Nichols)

**Objective:** Generic digital PID controller in VHDL with FPGA-based Ziegler-Nichols auto-tuning relay oscillation method.

**PID difference equation (Q12.4 fixed-point):**
```
u[n] = Kp*e[n] + Ki*Σe + Kd*(e[n]-e[n-1])
```

**Anti-windup:** Back-calculation method — subtract excess integral when output saturates.

**Auto-tune:** FPGA applies relay (bang-bang) control, measures ultimate period Tu and ultimate gain Ku from oscillation, computes Z-N gains, stores to registers.

**Testbench:** Plant model (first-order + delay) simulated in ModelSim; verify step response meets 25% overshoot, <2s settling.

---

### Project 41 — Real-Time Spectrum Monitoring & Anomaly Detection

**Objective:** Combine Projects 28 (FFT) and 40 (PID-like control) — continuously monitor a signal spectrum, detect anomalies (unexpected spectral peaks), trigger FPGA alarm output within 100 µs.

**VHDL anomaly detector:**
- Baseline spectrum stored in M9K RAM (updated with exponential moving average)
- Each new FFT bin compared: `if |bin - baseline| > threshold → alarm`
- Threshold: configurable per-bin (64 independent thresholds)

**Response:** FPGA GPIO alarm → optocoupler → external system (within 100 µs of anomaly entering ADC).

---

## DOMAIN 11 — CONFIGURATION MANAGEMENT & DOCUMENTATION
*Maps to: "Configuration and version control of the baselines", documentation requirements*

---

### Project 42 — FPGA Configuration Manager (Remote Bitstream Update)

**Objective:** Implement FPGA remote update mechanism — Pi 5 can load a new bitstream over SPI without physical access, using Cyclone III Remote Update megafunction.

**Architecture:**
```
Pi 5 (new bitstream) → SPI flash (EPCS16) ← [ FPGA Remote Update Controller ]
                                                      |
                                         [ AS (Active Serial) configuration ]
                                                      |
                                              [ Fallback image management ]
```

**VHDL Remote Update integration:**
```vhdl
-- Cyclone III ALTREMOTE_UPDATE megafunction
remote_update_inst : altremote_update
  generic map (
    OPERATION_MODE   => "REMOTE",
    RECONFIG_ENABLE  => "TRUE"
  )
  port map (
    clock            => clk_50,
    reset            => rst_n,
    reconfig         => reconfig_trigger,  -- from Pi 5 register
    param            => reconfig_addr,
    read_source      => read_src,
    busy             => busy_flag
  );
```

**Safety:** CRC-verified bitstream, rollback to factory image on boot failure (3 failed attempts).

---

### Project 43 — Hardware Description Requirements Management Tool

**Objective:** Pi 5 hosts a Flask-based web app that maps VHDL modules to requirements (like IBM DOORS, but lightweight), tracking implementation status.

**Database schema (SQLite):**
```sql
CREATE TABLE requirements (
  id TEXT PRIMARY KEY,   -- e.g. "REQ-UART-001"
  description TEXT,
  verification_method TEXT,  -- "simulation" | "hardware_test" | "inspection"
  status TEXT  -- "open" | "implemented" | "verified" | "closed"
);

CREATE TABLE vhdl_modules (
  module_name TEXT PRIMARY KEY,
  file_path TEXT,
  req_ids TEXT  -- comma-separated req IDs
);

CREATE TABLE test_results (
  req_id TEXT REFERENCES requirements(id),
  test_date TEXT,
  result TEXT,  -- "pass" | "fail"
  evidence_path TEXT
);
```

**Features:** Traceability matrix export (PDF), requirement coverage dashboard, automated test linkage.

---

### Project 44 — Automated Regression Test Runner (ModelSim + Python)

**Objective:** Python script orchestrates ModelSim batch simulations for all VHDL projects, collects pass/fail, generates HTML test report.

**Script architecture:**
```python
# run_regression.py
import subprocess, json, datetime

TEST_CASES = [
    {"name": "fifo_tb",    "do_file": "sim/fifo_tb.do"},
    {"name": "uart_tb",    "do_file": "sim/uart_tb.do"},
    {"name": "crc_tb",     "do_file": "sim/crc_tb.do"},
    # ... all 50 project testbenches
]

def run_modelsim(do_file):
    result = subprocess.run(
        ["vsim", "-batch", "-do", do_file],
        capture_output=True, text=True, timeout=300
    )
    passed = "SIMULATION PASSED" in result.stdout
    return {"passed": passed, "log": result.stdout}

results = [{"name": tc["name"], **run_modelsim(tc["do_file"])}
           for tc in TEST_CASES]
generate_html_report(results, f"report_{datetime.date.today()}.html")
```

---

## DOMAIN 12 — ADVANCED INTEGRATION PROJECTS
*Maps to: "Hardware bring-up (testing and debugging)", fault finding, full system integration*

---

### Project 45 — Multi-Board Synchronisation (Deterministic Latency)

**Objective:** Synchronise two FPGA boards to <10 ns latency using a shared reference clock and IEEE 1588 PTP-like timestamp exchange over Ethernet.

**FPGA timing slave:**
- Receives PTP event messages over UDP (Pi 5 as PTP master)
- Measures offset: `offset = (t2 - t1 - (t4 - t3)) / 2`
- Adjusts local 48-bit timestamp counter by computed offset

**Hardware:** 10 MHz OCXO as master reference → LVDS fanout → both FPGA boards.

**Verification:** Measure sync pulse skew between boards on oscilloscope; target <10 ns.

---

### Project 46 — Electronic Warfare Signal Emulator (Radar Pulse Modulator)

**Objective:** FPGA generates radar-like pulsed RF control signals (not RF itself — FPGA drives an IQ modulator or pulse shaper), including PRF, pulse width, stagger patterns.

**Pulse descriptor word (PDW) format:**
```vhdl
type pdw_t is record
  pri         : unsigned(23 downto 0);  -- pulse repetition interval, µs
  pulse_width : unsigned(15 downto 0);  -- µs
  frequency   : unsigned(31 downto 0);  -- Hz (to DDS)
  amplitude   : unsigned(11 downto 0);  -- DAC code
  modulation  : std_logic_vector(3 downto 0);  -- CW/FM/PM/AM
end record;
```

**PDW sequencer:** Reads PDWs from M9K RAM, generates timing triggers for DAC and RF switch control GPIOs.

**Stagger patterns:** 2/3/4-position PRI stagger with jitter injection (±5% random via LFSR).

---

### Project 47 — Secure Boot & Bitstream Encryption

**Objective:** Implement Cyclone III AES-128 bitstream encryption with a device-unique key, and add an FPGA-side authentication challenge using HMAC-SHA1 (implemented in VHDL).

**Key storage:** Cyclone III battery-backed key registers (KEY[127:0]) — load via JTAG in secure facility.

**FPGA authentication engine:**
- 128-bit challenge from Pi 5 over SPI
- VHDL SHA-1 core computes HMAC
- 160-bit response sent back within 10 ms
- Pi 5 verifies; if mismatch → lock UART/SPI interfaces via FPGA register

**VHDL SHA-1 core:** Implement 80-round iterative SHA-1 in VHDL, targeting <1000 LEs on EP3C16.

---

### Project 48 — Built-In Self-Test (BIST) Controller

**Objective:** FPGA autonomously runs start-up BIST on all its internal memories (M9K march test), logic (scan chain), and I/O buffers before declaring system ready.

**March test for M9K BRAM:**
```vhdl
-- March C- algorithm
-- Phase 0: ↑ (w0)   — write 0 ascending
-- Phase 1: ↑ (r0,w1) — read 0, write 1 ascending
-- Phase 2: ↑ (r1,w0) — read 1, write 0 ascending
-- Phase 3: ↓ (r0,w1) — read 0, write 1 descending
-- Phase 4: ↓ (r1,w0) — read 1, write 0 descending
-- Phase 5: ↓ (r0)    — read 0 descending
```

**JTAG scan chain:** Use Cyclone III JTAG interface to shift test pattern through boundary scan chain; compare with expected response.

**BIST report:** 8-bit result code transmitted via UART; LED indicator for pass/fail.

---

### Project 49 — Full System Integration: Saab-Inspired Signal Processing Chain

**Objective:** Integrate Projects 7 (ADC), 17 (Laser driver), 18 (Lock-in amp), 28 (FFT), 30 (DSO), 36 (Data logger) into a coherent multi-channel signal acquisition and analysis system.

**System architecture:**
```
[ Laser Driver (Proj 17) ] → [ Optical Target ] → [ TIA (Proj 16) ]
                                                         |
                                                   [ ADC 12-bit ]
                                                         |
                                                 [ FPGA EP3C16 ]
                                                    /    |    \
                                           Lock-In  FFT  DSO  Logger
                                           (P18)   (P28) (P30) (P36)
                                                         |
                                                      [ Pi 5 ]
                                                    (display, DB, alert)
```

**FPGA resource utilisation report (target for EP3C16F484C6N):**
| Module | LEs | M9K blocks | Multipliers |
|--------|-----|------------|-------------|
| ADC capture | 200 | 2 | 0 |
| Lock-in amp | 800 | 4 | 8 |
| 64-pt FFT | 1200 | 6 | 16 |
| DSO trigger | 400 | 4 | 0 |
| Data logger | 300 | 2 | 0 |
| UART/SPI I/F | 500 | 2 | 0 |
| **Total** | **3400** | **20** | **24** |
| **EP3C16 capacity** | **15408** | **56** | **56** |
| **Utilisation** | **22%** | **36%** | **43%** |

**Documentation deliverables:** System requirements document, architecture design description, test procedure, test report — following hardware development process.

---

### Project 50 — Digital Beamforming Simulation (4-Element Phased Array)

**Objective:** Implement a 4-element phased array beamformer in VHDL — complex phase-shift application, coherent summation, beam steering angle control — directly applicable to Saab EW/radar systems.

**Architecture:**
```
ADC0 → [ Phase Shift (θ0) ] ─┐
ADC1 → [ Phase Shift (θ1) ] ─┤
ADC2 → [ Phase Shift (θ2) ] ─┼── [ Complex Adder ] → [ Beam Output ]
ADC3 → [ Phase Shift (θ3) ] ─┘
         ↑
   [ Beam Steering Controller ]
   Angle → phase word per element
   Δφ = (2π/λ) × d × sin(θ)
```

**VHDL phase shifter:**
```vhdl
entity complex_phase_shift is
  generic (DATA_W : integer := 16);
  port (
    clk       : in  std_logic;
    data_re   : in  signed(DATA_W-1 downto 0);
    data_im   : in  signed(DATA_W-1 downto 0);
    phase_word: in  signed(15 downto 0);  -- Q1.15 cosine/sine lookup index
    out_re    : out signed(DATA_W downto 0);
    out_im    : out signed(DATA_W downto 0)
  );
  -- Implements: out = in × e^(jφ) = in_re×cos(φ) - in_im×sin(φ)
  --                                    in_re×sin(φ) + in_im×cos(φ)
end entity complex_phase_shift;
```

**Beam pattern verification:** Simulate 4-element ULA with λ/2 spacing; sweep beam from -60° to +60°; verify 3-dB beamwidth ≈ 25° and first sidelobe < -13 dBc.

**Resource usage:** 4 phase shifters × 4 multipliers each = 16 multipliers; well within EP3C16's 56 multipliers.

---

## APPENDIX A — Quartus Prime Project Template

```
project/
├── rtl/
│   ├── lib/          ← reusable IP (Projects 1-5)
│   ├── dsp/          ← filter/FFT/DDS
│   ├── comms/        ← UART/SPI/I2C/CAN/HDLC
│   ├── analog_ctrl/  ← ADC/DAC/VGA/TIA control
│   └── top/          ← system integration
├── sim/
│   ├── tb/           ← testbench files
│   ├── scripts/      ← ModelSim .do scripts
│   └── reports/      ← coverage, timing reports
├── constraints/
│   ├── *.qsf         ← Quartus settings
│   └── *.sdc         ← timing constraints
├── docs/
│   ├── requirements/ ← requirements documents
│   ├── design/       ← design descriptions
│   └── test/         ← test procedures & reports
└── ltspice/          ← analog simulation files
```

---

## APPENDIX B — ModelSim Simulation Script Template

```tcl
# Generic simulation script — sim/run_sim.do
vlib work
vmap work work

# Compile library
vcom -2008 -work work ../../rtl/lib/fifo/generic_fifo.vhd
vcom -2008 -work work ../../rtl/lib/uart/uart_top.vhd
vcom -2008 -work work ./tb/uart_tb.vhd

# Simulate
vsim -t 1ps -novopt work.uart_tb

# Waveforms
add wave -divider "UART TX"
add wave /uart_tb/dut/tx_pin
add wave /uart_tb/dut/tx_data
add wave -divider "UART RX"
add wave /uart_tb/dut/rx_data
add wave /uart_tb/dut/rx_valid

run 1 ms
coverage report -detail
quit -code [expr {[coverage get -assert failures] > 0}]
```

---

## APPENDIX C — Cyclone III EP3C16F484C6N Resource Summary

| Resource | Available | Notes |
|---|---|---|
| Logic Elements (LEs) | 15,408 | Combinatorial + register |
| Embedded Memory (M9K) | 56 blocks × 9Kbit | 504 Kbit total |
| 18×18 Multipliers | 56 | DSP blocks |
| PLLs | 4 | Up to 550 MHz VCO |
| User I/O | 346 | LVTTL, LVDS, SSTL, HSTL |
| LVDS pairs | 133 | RX pairs |
| Package | 484-pin FBGA | 23×23 mm |
| Configuration | EPCS16 via AS | 16 Mbit serial flash |
| JTAG | Yes | SignalTap II logic analyser |

---

## APPENDIX D — Test Documentation Template (per project)

### Test Procedure: `[Project Name]`
**Requirement ID:** REQ-XXX-YYY  
**Test Method:** Simulation / Hardware / Inspection  
**Pass Criteria:** [specific, measurable criterion]  

| Step | Action | Expected Result | Actual Result | Pass/Fail |
|------|--------|----------------|---------------|-----------|
| 1 | Apply stimulus X | Output Y within Z | | |
| 2 | ... | ... | | |

**Test Equipment:** ModelSim-Altera, Quartus Prime Lite, Oscilloscope, Logic Analyser  
**Test Date:** ___________  **Engineer:** ___________  **Signature:** ___________

---

*Document prepared for Saab Grintek Defence Hardware Design Engineer role evaluation*  
*Platform: Intel/Altera Cyclone III EP3C16F484C6N + ESP32 + Raspberry Pi 3B/5*  
*Toolchain: Quartus Prime Lite, ModelSim-Altera, LTSpice, KiCad*
