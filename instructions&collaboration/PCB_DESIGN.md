# ⚡ PCB Design & Hardware Electronics Guidelines

> **Mandatory Team Policy:** All electronics and PCB design team members must strictly comply with these hardware design rules, safety margins, review protocols, and bring-up procedures before any board is approved for fabrication or connected to a battery.

---

## 1. Scope & Objectives

The Xerox UAV PCB design sub-team is responsible for:
- Custom Power Distribution Boards (PDB) and smart battery management interfaces
- Flight controller companion boards and microcontroller shields
- Sensor breakout boards (IMU, Barometer, Magnetometer, Optical Flow, ToF LiDAR)
- Communication, telemetry, and payload actuation carrier boards

Electronics failures on a quadcopter lead directly to catastrophic physical crashes and battery fires. High rigor, redundancy, and defensive design are strictly enforced.

---

## 2. Tools & Component Library Standards

1. **Approved EDA Suites:**
   - **KiCad 8.0+** (Primary open-source standard for the team).
   - **Altium Designer** (Permitted when project lead specifies).
2. **Component Libraries:**
   - Always verify custom footprints against official manufacturer component datasheets (pin numbering, pad dimensions, courtyard clearances, pin 1 orientation).
   - Include clear 3D STEP models for all components to enable CAD clearance checks with the airframe mechanical team.
3. **Component Sourcing & BOM:**
   - Standardize components around readily available distributors (JLCPCB SMT parts library, LCSC, Digi-Key, Mouser).
   - Clearly document Manufacturer Part Number (MPN) and package size (prefer 0603 or 0805 for passives to facilitate manual rework).

---

## 3. Schematic Design Principles

- **Hierarchical Sheets:** Divide schematics into clean functional blocks (e.g., `Power Regulation`, `MCU Core`, `Sensors & Filtering`, `Connectors & I/O`).
- **Power Rail Naming:** Explicitly label power nets with voltages and domain types:
  - `VBAT_RAW` (Direct battery voltage)
  - `+5V0_CLEAN` (Buck regulator output for sensors/logic)
  - `+3V3_MCU` (LDO output dedicated to microcontroller)
  - `+5V0_SERVO` (High-current rail for servos/actuators)
- **Decoupling Capacitors:**
  - Place at least one 100nF ceramic decoupling capacitor per IC power pin, placed directly next to the pin.
  - Include bulk capacitance (10µF–100µF low-ESR ceramic or tantalum) at the output of every voltage regulator and battery entry.
- **Protection Circuits (Mandatory):**
  - **Reverse Polarity Protection:** P-channel MOSFET or ideal diode circuit on all external power inputs.
  - **TVS Diodes:** Place Transient Voltage Suppression (TVS) diodes (e.g., SMAJ series) on main battery leads to suppress inductive voltage spikes during motor braking / active freewheeling.
  - **Fusing:** Resettable PPTC polyfuses on 5V accessory and telemetry rails to prevent single-sensor shorts from bringing down the flight computer.
  - **ESD Protection:** ESD arrays on all external data ports (USB, CAN, UART, I2C).

---

## 4. PCB Layout & Routing Guidelines

### 4.1. Layer Stackup
- **4-Layer Minimum** for all mixed-signal or microcontroller boards:
  - Layer 1 (Top): High-speed signals & component placement
  - Layer 2 (Inner 1): Continuous solid Ground Plane (`GND`)
  - Layer 3 (Inner 2): Power planes & secondary signal routing
  - Layer 4 (Bottom): Ground plane & secondary components

### 4.2. High-Current & Power Routing
- **Copper Thickness:** Specify **2 oz copper (70 µm)** for PDBs and motor drive boards.
- **Trace Width Calculation:** Size high-current traces according to IPC-2152 based on expected continuous and burst motor currents.
- **Polygon Pours & Stitching:** Use wide copper pours for battery positive and ground paths. Stitch copper pours with dense via matrices (via diameter 0.3mm–0.5mm) to distribute thermal dissipation and reduce inductance.

### 4.3. Noise & EMI Mitigation
- **Physical Isolation:** Strictly separate noisy high-current switching circuits (buck converters, ESC motor power lines) from sensitive analog circuits (magnetometer, IMU, barometric pressure sensors, GPS RF frontend).
- **Ground Loops:** Maintain an unbroken ground return path directly under high-speed digital tracks (SPI, CAN, RMII). Do not route signals across splits in the ground plane.
- **Switching Inductors:** Keep the high di/dt loop of buck regulators as small as possible. Place the inductor, input capacitor, and diode/MOSFET immediately adjacent to each other.

### 4.4. Connector Standardization
| Interface | Standard Connector Type |
| :--- | :--- |
| Main Battery In | XT60 (Large Quad) / XT30 (Micro/Mini Quad) |
| High-Speed CAN / DroneCAN | JST-GH 4-pin |
| Telemetry & GPS UART | JST-GH 6-pin |
| I2C Sensors | JST-GH 4-pin |
| Low-Profile Inter-board | Molex PicoBlade or 0.5mm FPC |

---

## 5. Design Rule Check (DRC) & Verification

Before submitting design files:
1. Run **Electrical Rules Check (ERC)** on schematics: Zero warnings, zero errors.
2. Run **Design Rules Check (DRC)** in PCB Layout matching the targeted manufacturer’s specs:
   - Minimum trace width / spacing (typically 6mil / 6mil or 5mil / 5mil)
   - Minimum drill size & annular ring
   - Solder mask expansion & silkscreen clearances
3. Verify mechanical dimensions and mounting hole clearances against CAD drawings. Ensure mounting holes support standard M2 / M3 vibration dampening rubber standoffs.

---

## 6. Manufacturing Export & Safe Bring-Up Procedure

### 6.1. Release Package Requirements
When opening a release PR for a board, include in the repository release or `hardware/build/` folder:
- Complete Gerber files (RS-274X format)
- NC Drill files
- Bill of Materials (`BOM.csv`) with supplier reference codes
- Centroid / Pick-and-Place file (`positions.csv`)
- High-resolution schematic PDF

### 6.2. Physical Board Bring-Up Protocol
Never connect a LiPo battery directly to a newly soldered board! Follow this exact checklist:
1. **Visual Inspection:** Inspect under microscope for solder bridges, cold joints, reversed diode/tantalum capacitor polarities.
2. **Resistance Check (Multimeter):** Measure resistance between `VBAT` and `GND`, and between each regulated rail (`+5V`, `+3V3`) and `GND`. Ensure there is NO short circuit.
3. **Current-Limited Bench Supply:** Power the board from a bench supply set to target voltage with current limit capped at **100 mA**.
4. **Voltage Rail Verification:** Measure all regulated voltage test points before connecting microcontrollers or peripherals.
5. **Sensor Smoke Test:** Incrementally connect sensors one-by-one while monitoring supply current.
