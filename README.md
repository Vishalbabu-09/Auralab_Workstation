# AURALAB WORKSTATION: Pocket-Sized Lab Bench & Hardware Diagnostic Toolkit

An advanced, ultra-portable multi-function test, measurement, and diagnostic platform engineered for the **KiCad PCB Design Contest**.

The **Auralab Workstation** packs a 12-bit digital oscilloscope, a variable bipolar $\pm15\text{V}$ bench power supply, an automated component curve tracer, and a smart hardware safety fault interrupter into a highly integrated, pocket-sized form factor.

---

## 1. System Working Modes & Operational Logic

The workstation utilizes the dual-core architecture of the **RP2040 microcontroller** to handle high-speed telemetry ingestion on one core while managing the graphical user interface (UI) and safety control loops on the second core.

### 📈 Mode 1: Oscilloscope Mode
* **What it does:** Captures, visualizes, and measures high-frequency external electrical signals in real time directly on the integrated 2.4" TFT display.
* **How it works:** Raw signals pass through an AC/DC coupling selector switch into a high-impedance **TL072 JFET operational amplifier** buffer circuit. This prevents the workstation from loading down the circuit under test. The conditioned signal is digitized by a dedicated **MCP3201 12-bit SAR ADC** operating over a high-speed SPI bus. Core 0 of the RP2040 processes the raw data arrays to compute signal parameters including **frequency, duty cycle, time period, $V_{pp}$ (peak-to-peak), and $V_{RMS}$**.

### ⚡ Mode 2: Bipolar DC Bench Power Supply Mode
* **What it does:** Acts as a precision-adjustable split-rail power supply delivering up to $\pm15\text{V}$ to drive external analog, op-amp, and prototype circuits via low-impedance screw terminals (`J2`).
* **How it works:** The user rotates the incremental digital encoder to set a target voltage. The RP2040 transmits this target via SPI to an **MCP42010 Dual Digital Potentiometer**. This digital pot dynamically alters the resistor feedback network of a high-efficiency **TPS65131 split-rail buck-boost regulator**, scaling the outputs symmetrically.

### 📊 Mode 3: Automated Component Curve Tracer Mode
* **What it does:** Plots the characteristic Voltage vs. Current ($V\text{-}I$) curves of un-biased two-terminal components (diodes, transistors, resistors, LEDs) directly onto the display to identify unknown parts or detect faulty silicon behavior.
* **How it works:** The workstation generates an internal hardware voltage sweep using a smoothed, high-frequency **Sweep PWM signal**. This low-pass filtered signal is amplified through a dedicated stage of the **TL072 op-amp**. As the voltage sweeps across the test terminal, an **INA219 high-side current monitor** reads the voltage drop across a $0.1\Omega$ shunt resistor, graphing the exact threshold voltages ($V_{th}$), saturation curves, and breakdown regions.

### 🛑 Mode 4: Smart Fault Detection & Overload Interrupter
* **What it does:** Acts as a real-time, hybrid hardware-software circuit breaker protecting both the workstation and your external device under test from short circuits, overcurrent conditions, and reverse polarity.
* **How it works:** The **INA219 current sensor** constantly monitors active current draw. If the current spikes past a safe user-defined ceiling, a hardware interrupt flag (`CURRENT_ALERT`) fires. The microcontroller instantly isolates the output terminals, flashes a high-intensity **ALERT_LED**, and triggers a resonant **Piezo Buzzer** to notify the user. Backup physical protection is maintained via an inline **Polyfuse (F1)** and steering clamping diodes.

---

## 2. Advanced 4-Layer PCB Stackup & Plane Allocation

To preserve 12-bit ADC accuracy while operating high-frequency digital clocks right next to high-gain analog buffers, this board utilizes a strict 4-layer mixed-signal stackup.

| Layer | Name | Copper Type | Main Signals & Plane Assignments |
| :--- | :--- | :--- | :--- |
| **Layer 1** | `F.Cu` (Top) | Signal / High-Speed | Component placement, SPI buses (Display, Flash, ADC), $\text{I}^2\text{C}$ lines, and sensitive analog trace lines. |
| **Layer 2** | `In1.Cu` | Split Ground Plane | **`GND` (Digital Ground Plane)** under the RP2040/Flash.<br>**`AGND` (Analog Ground Plane)** under the Op-Amp/ADC/Reference.<br>*Isolated by a physical gap and joined strictly at a single point via a `NetTie_2` footprint near the power entrance.* |
| **Layer 3** | `In2.Cu` | Power Plane | Solid power routing zones split cleanly into **`+3V3_DIGITAL`**, **`+3V3_ANALOG`**, and **`+5V_RAW`** copper pours to eliminate voltage drops. |
| **Layer 4** | `B.Cu` (Bottom) | Signal / Return Shield | Low-frequency control signals (Encoder pins, switch tracking) and secondary ground fills for via stitching shields. |

---

## 3. Detailed Schematic & Hardware Architecture Breakdown

The hardware layout is organized into three highly optimized hierarchical sheets:

### 🔋 Module 1: Power Management & Filtration (Sheet 2)
Handles the regulation of raw input vectors, lithium-ion battery management, and high-current variable power rails.
* **USB-C Interface (`J3`):** Employs a 16-pin USB-C receptacle protected by an ultra-low capacitance **USBLC6-25C6 ESD array** to prevent electro-static damage across the differential lines.
* **BMS Stage (`U8`):** Uses a **TP4056 linear charger** with a precision $1.2\text{k}\Omega$ $R_{prog}$ resistor to regulate charging cycles safely for an attached Li-Po battery.
* **Split-Rail Engine (`U5`):** The **TPS65131 buck-boost converter** functions alongside high-current **SRR6048** power inductors ($L1, L2$) and high-speed Schottky catch diodes to cleanly establish the variable bipolar tracking rails.

### 🧠 Module 2: RP2040 Core & Interface Subsystem (Sheet 3)
Formulates the digital brain, memory maps, and human-machine interface routing.
* **Clocking Engine (`Y1`):** A low-jitter **12MHz Quartz Crystal** ensures ultra-stable operation for synchronous SPI data ingestion from the high-resolution ADC.
* **Storage Array (`U11`):** A **W25Q128JVS 128Mb QSPI Flash IC** stores custom GUI frames, display fonts, and analytical logging tables.

### 🔬 Module 3: Precision Analog Front-End (Sheet 4)
Handles conditioning, front-end buffering, clamping, and current telemetry tracking.
* **Precision Voltage Reference (`U3`):** Uses a low-drift **REF3033 bandgap voltage reference** to output a rock-solid, noise-free $3.3\text{V}$ absolute ceiling (`Vref_ADC`) directly to the SAR ADC, completely mitigating reference measurement drift.
* **Dynamic Range Shifter (`Q1`):** Features an automated hardware range shifter. By steering the gate of an **85170 N-Channel Power MOSFET**, the microcontroller alters the shunt resistor topology dynamically, switching scales between high-current tracking and micro-amp precision modes automatically.

---

## 4. Manufacturing & STEP Mechanical Export Validation

The project layout files have passed full Design Rule Checking (DRC) with 0 errors. A universal 3D mechanical master assembly model has been generated directly from the layout engine:

* **Export Format:** STEP (.step) standard mechanical vector.
* **Output Package Verified:** `Auralab_Workstation_Vishal.step` includes fully rendered 3D models for all high-profile custom footprints, including the 2.4" TFT module, the **XY126-3P** green terminal blocks, and the shielded **SRR6048** power inductors.
* **Compatibility:** Fully compliant with industry standard CAD software (SolidWorks, Autodesk Fusion 360, and free web viewers like `viewer.autodesk.com`).

---

## 5. Repository Directory Tree

```text
├── 3D Models Auralab_Workstation_Vishal/  <- Local STEP structural component cache
├── Auralab_Workstation Images/            <- Visual project logs and board renderings
├── Auralab_Workstation_Vishal schematic/  <- Schematic sheet files
├── Manufacturing/
│   ├── Drill/                             <- NC Drill files (.drl)
│   └── Gerbers/                           <- RS-274X Production plots (.gbr)
├── Auralab_Workstation_Vishal.kicad_pro   <- Master KiCad Project Hub File
├── Auralab_Workstation_Vishal.kicad_sch   <- Master Hierarchical Schematic Sheet
├── Auralab_Workstation_Vishal.kicad_pcb   <- Complete 4-Layer Routed PCB Layout
├── Auralab_Workstation_Vishal.pdf         <- Complete schematic printable export
└── LICENSE                                <- Open-source distribution documentation
