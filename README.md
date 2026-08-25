# PC-Guided Wire Harness Sub-Assembly Station
## Market research, working principles, and a build strategy

Prepared 25 August 2026 · Based on plant photographs (Toyota Yaris harness line, Karachi) and open-source industrial/patent research

---

# PART 1 — WHAT THIS MACHINE ACTUALLY IS

## 1.1 Reading your existing machine

The label on the bench in your photo says **"PC GUIDED SUB-ASSY MACHINE"**. That is the correct, standard name for this class of equipment. In the wider industry it appears under several names:

| Name in use | Where you'll hear it |
|---|---|
| PC-Guided Sub-Assy Machine | Japanese-lineage plants (Yazaki, Sumitomo, TE Japan lines) |
| Sub-Assy Navigation System / Assembly Navigation | Yazaki, Sumitomo internal terminology |
| Guided Assembly / Build-Aid Tester | Cirris, CAMI Research (US) |
| Terminal Insertion Guiding Apparatus | Patent literature (Sumitomo Wiring Systems) |
| Wiring Harness Poka-Yoke Machine | Indian SPM builders |
| Operator Guidance Platform | Arkite, LightGuide (projected-AR class) |

**Decoding the screen you photographed:**

```
A-2551 ST-7.20          <- station / line ID
Page No.:      33       <- step 33 of the build sequence
Wire Cabin:    0        <- wire feed rack / cabin position
Job Card No.:  2488     <- production order (traceability key)
Circuit No.:   YQ4      <- circuit designator from the harness drawing
Wire Gauge:    0.35     <- mm² (AVSS 0.35)
Wire Color:    L/       <- L = blue (JIS colour code), "/" = no stripe
Wire Length:   1512     <- mm
[progress bar with connector icon at each end]
A B [D] E F G           <- cavity letters; D highlighted = insert HERE now
ASZLE6-35F   |  SSGTSN8-18M-L   <- the two connector part numbers
[Checker] [SB Start] [NG]       <- operator control buttons
```

The "Possible Wiring Harness Defects" board next to it references **"Dotsu NG"**. *Dōtsū* (導通) is Japanese for **electrical continuity**. So your current machine is not just a screen — it is doing a **live continuity check** on every insertion, and "Dotsu NG" means the continuity probe did not see the terminal seat. That single word tells you the architecture family the machine belongs to (see §1.3, Family B).

The six defects the board is trying to catch:

1. Circuit wire miss (wire not inserted at all)
2. Wrong wire insertion vs. drawing call
3. Retainer miss / open (TPA not closed)
4. **Terminal backout — TBO / NP lock** (terminal not locked; backs out on pull check)
5. Accessory part missing (corrugated tube, vinyl tape, grommet, sleeve)
6. Male terminal pin bend (up/down)

Note that only #1, #2 and #4 are electrically detectable. #3, #5 and #6 need mechanical or optical sensing. This distinction drives the whole sensor architecture — I come back to it in §3.6.

## 1.2 Why the operation resists automation

The core insight from the industry literature (CAMI Research's white paper on computer-guided harness assembly puts it well): inserting a terminated wire into a connector cavity depends on human vision and finger dexterity in a way no cost-effective machine matches. Every serious system in this category therefore **assists the human** rather than replacing them, targeting three goals:

1. Reduce the abstraction level of pin location (don't make the operator map "cavity D" to a physical hole)
2. Minimise physical motion to insert a pin
3. Provide positive feedback that insertion is complete and correct

Four perceptual burdens are removed: identifying the pinned wire, looking its code up in a build list, finding the target cavity in a crowded housing, and confirming the insertion. Field data from these systems shows **20–50% productivity improvement with error rates driven to near zero**.

## 1.3 The four architecture families

### Family A — Cavity illumination (light-per-cavity)
A fixture holds the connector. A super-bright LED drives a plastic optical fibre that terminates *inside* each cavity, so the target cavity glows and flashes. When the terminal is inserted it physically blocks the light — insertion confirmation is free. Unpopulated cavities can all be flashed at once to guide seal-plug insertion.

- **Pro:** eyes stay on the work, not on a monitor. Self-confirming. Works for first-sided pinning (no far-end connector needed).
- **Con:** one fibre-loaded fixture per connector part number. That fixture library is the dominant cost.
- **Reference:** CAMI Research "Light Director"; Cirris fixturing with guidance LEDs.

### Family B — Continuity / second-side electrical sensing ← *your machine*
The connector under assembly sits in a receiver jig with a **spring probe (pogo pin) at the back of every cavity**. The far end of the wire is either already in a mated fixture, or the operator touches the loose wire to a probe/wrist-strap contact. The controller drives a signal down the wire and watches which probe sees it.

Two things fall out of this for free:
- **Wire identification by touch.** Touch a wire, the system knows which circuit it is, and lights/announces the target cavity. No barcode, no typing.
- **Insertion verification.** Continuity appears at the expected probe = OK, at a different probe = wrong cavity, nowhere = not seated (that is your "Dotsu NG").

Sumitomo's patent literature (US 6,993,833, "Apparatus for manufacturing sub-harness") describes exactly this loop: terminal contacts the probe → continuity checked against the second connector → NG blocks advancing to the next terminal and fires a warning lamp and buzzer → OK advances the guide to the next cavity → completion button counts the unit and releases it.

Earlier prior art (US 5,477,606) notes the weakness of rigid per-connector receptors: each connector shape needs its own receptor, which is costly, bulky, and slow to change over. This is *the* design problem you must solve.

- **Pro:** catches wrong-cavity and not-seated defects with certainty. Extremely fast once fixtured.
- **Con:** needs a probe per cavity and a mating fixture per connector.

### Family C — Screen + pick-to-light (no per-cavity feedback)
A monitor shows the connector graphic with the target cavity highlighted; connector bins and wire rack positions are indicated with lamps; the operator confirms each step with a foot pedal or button. Cheap, universal, no connector-specific fixture — but it is *guidance only*, not verification. A determined mistake still gets through.

### Family D — Projected AR / vision
An overhead high-lumen projector paints 1:1 CAD geometry, routing paths and target callouts directly onto the board; a 3D sensor or camera validates each step and refuses to advance until it is correct ("no-faults-forward"). LightGuide and Arkite are the two names to know. Excellent for the **layout board** stage (your second and third photos — the big yellow board with the clamps), less good for the fine-grained cavity work.

- **Pro:** zero connector-specific fixturing; handles high variant counts; full traceability.
- **Con:** highest cost per station; struggles to resolve a 2.2 mm cavity; ambient light and occlusion by hands.

### Practical conclusion
For a Toyota/Hyundai-grade sub-assy line you want **B as the backbone, A layered on top for high-mix connectors, C as the fallback for connectors too odd to fixture, and D reserved for the layout/taping board.** Design the controller so all four modes are supported by the same software and step file.

---

# PART 2 — MARKET, VENDORS AND COST

## 2.1 Who sells what

| Vendor | Product | Family | Notes |
|---|---|---|---|
| **Cirris** (US) | 8100 / CH2 low-voltage testers + Easy-Wire software | B (+A) | 256 test points base, expandable in 256-pt increments to 100,000. "Smart-Lights" adapters carry an ID chip so a fixture can be plugged into *any* tester port and is auto-recognised. Guided assembly with on-screen connector images, board LEDs and audible prompts. Also does end-of-line test on the same board. |
| **CAMI Research** (US) | CableEye + AutoBuild + Light Director | A | Fibre-into-cavity illumination; the published cost model is the most transparent in the industry (see below). |
| **Komax** (CH) | Zeta / harness automation | Full automation | The only vendor that automates insertion itself, with vision-guided Cartesian robots. Different budget universe. |
| **Schleuniger / Metzner** | Wire processing upstream | — | Cut/strip/crimp; feeds your sub-assy station. |
| **LightGuide** (US) | Projected AR platform | D | 200+ deployments; MES/PLC/camera integration. |
| **Arkite** (BE) | Operator Guidance Platform | D | Explicitly markets wire-harness routing via 1:1 CAD projection with 3D-sensor validation. |
| **Purvi Automation, and similar Pune/Chennai SPM builders** | "Wiring Harness Poka-Yoke Machine" | B/C | Regional, custom-built, price-competitive — your realistic commercial benchmark. |
| **Hefei Better, Suzhou Sanao, Dongguan builders** | Housing insertion machines | Automation | Automatic housing insertion for simple 2–12 pin housings, not multi-branch sub-assy. |

## 2.2 Reference cost points (verified where possible)

CAMI publishes a worked example that is worth internalising because it sets the shape of the economics:

- 256-point test platform + AutoBuild software + voice: **~USD 3,400** one-time, 5-year depreciation → about **USD 75/month per station** including calibration and extended warranty
- **~USD 450 per Light Director fixture board** (board kit, LED fibres, mating connector, assembly labour, programming)
- With 25–40% labour saving, two fixtures (USD 900) pay back in **14–23 working days**, or **56–90 assemblies**

Order-of-magnitude for the other families (these are directional estimates from vendor positioning, not published price lists — treat as planning figures to be replaced by real quotes):

| Class | Indicative capital per station |
|---|---|
| Screen-only guidance (Family C), self-built | USD 800 – 1,500 |
| Continuity-verified station (Family B), self-built, 256 pts | USD 2,500 – 5,000 + jigs |
| Continuity-verified station, imported (Cirris-class) | USD 6,000 – 15,000 + jigs |
| Connector-specific jig (either family) | USD 250 – 600 each |
| Projected AR station (Family D) | USD 30,000 – 80,000 |
| Full automated insertion cell | USD 250,000+ |

**The single most important cost fact:** across every family except D, the electronics are cheap and the **connector-specific fixture library is where the money goes**. A plant running Yaris + Hyundai variants can easily need 60–150 distinct connector jigs. At USD 400 each that is USD 24,000–60,000 — several times the controller cost. Your design must attack fixture cost, or the project's economics are decided before you write a line of firmware.

## 2.3 Where the ROI actually comes from

Four buckets, in rough order of value for an automotive tier-1:

1. **Escaped-defect elimination.** A wrong-cavity terminal that reaches the OEM is a containment event. Wire harness assembly defects are a recurring recall cause. This is the number that gets the project funded.
2. **Rework and scrap reduction** at the checker station downstream.
3. **Labour and cycle time**, 20–50% on the insertion operation.
4. **Training and flexibility.** New operators productive in days instead of weeks; variant changeover becomes a job-card scan.

Add a fifth that management will value: **traceability**. Every insertion timestamped against job card 2488 gives you a defensible record per unit.

---

# PART 3 — SYSTEM DESIGN

## 3.1 Design principles

1. **Split real-time from presentation.** The PC owns the database, the Excel import, the graphics and the MES link. The MCU owns microsecond-scale scanning, LEDs, load cells and interlocks. Never make the PC responsible for a timing-critical loop.
2. **Job programs are data, not firmware.** A new harness must never require a firmware flash. Firmware OTA is a separate, rare, signed operation.
3. **Fixtures are self-identifying.** Every jig carries an ID chip. Plug it anywhere; the system knows what it is and where. This is the single highest-leverage feature — it is what makes Cirris's Smart-Lights valuable, and it kills the changeover problem in US 5,477,606.
4. **Fail closed.** NG must physically block advancement and require a supervisor-authorised clear, not a click-through.
5. **Degrade gracefully.** If the continuity hardware is unavailable, the station falls back to Family C screen guidance and flags the units as "guided but unverified" in the log.

## 3.2 Recommended hardware stack

### Station computer (HMI, database, Excel, MES)
**Fanless industrial mini-PC, Intel N100 / N150, 8–16 GB RAM, 256 GB SSD, dual display out, 2× GbE, 4× USB, 12–24 V DC input.**
Roughly USD 180–280. Examples: any of the widely available N100 fanless boxes; or a branded industrial unit (Advantech ARK, Beckhoff C6015) if the customer demands a nameplate.

Why an x86 PC and not a Pi:
- Your plant already runs Windows on these benches; IT support and spares in Karachi are trivial
- Excel/OpenXML parsing, SQL Server / PostgreSQL clients, and MES connectors are first-class
- Fanless N100 handles 1080p connector graphics and a projector output simultaneously
- Five-plus-year commercial availability

Run **Windows 11 IoT Enterprise LTSC** if the plant standard is Windows, otherwise **Ubuntu 24.04 LTS in kiosk mode**. Lock down USB, auto-login, watchdog-restart the HMI app.

*(Raspberry Pi CM5 on a carrier board is a legitimate cheaper alternative at ~USD 120 if you control the whole software stack and don't need Windows. Decide once, plant-wide.)*

### Station controller (real-time master)
**ESP32-S3 (dual-core, 240 MHz, 8 MB PSRAM) + W5500 or LAN8720 wired Ethernet**, on your own PCB.

- Why ESP32-S3: you already have deep ESP32 experience, mature OTA (`esp_ota` + rollback), cheap, excellent SPI/I²C/UART peripheral count, USB-OTG for local firmware recovery.
- **Use wired Ethernet, not Wi-Fi.** A harness plant is a steel-and-copper Faraday nightmare with fans, welders and a hundred stations. Keep Wi-Fi as commissioning-only.
- Alternative for production v2 if you want longer industrial lifecycle and hard determinism: **STM32H563 or STM32G474** with built-in Ethernet MAC. More work, better 10-year story, better ADC for load cells.

**Recommendation: prototype and pilot on ESP32-S3; keep the PCB footprint and firmware HAL abstract enough that an STM32 swap is a board respin, not a rewrite.**

### Jig nodes (one per connector fixture)
**RP2040 or STM32G031** + RS-485 transceiver (SN65HVD72 or MAX3485), addressed over a daisy-chained bus.

Each node provides:
- Up to 32 cavity channels (probe sense + LED drive)
- A **1-Wire DS28E07 or I²C EEPROM holding the fixture's identity** (connector P/N, cavity map, calibration, revision)
- Local LED current control and a status RGB indicator

Cost per node: USD 12–20 in parts at low volume.

**Bus choice: RS-485 running Modbus RTU at 500 kbps, or a lightweight custom framed protocol.** Rationale: cheap, noise-immune over 20 m of shop floor, daisy-chainable, no switch needed, and any PLC engineer in the plant can debug it. CAN-FD is the alternative if you want automotive-grade arbitration — defensible but heavier tooling.

## 3.3 The continuity scanning engine

This is the heart of Family B. Design it properly and everything else follows.

**Topology:** an N-point scanner where every point can be a driver or a sensor.

```
        ┌──────────── Station Controller (ESP32-S3) ────────────┐
        │  scan sequencer · netlist compare · step machine      │
        └───┬───────────────────────────────────────────────┬───┘
            │ SPI (drive)                     SPI (sense)   │
     ┌──────▼──────┐                          ┌─────────────▼─────┐
     │ One-hot drive│                          │ 16:1 analog mux   │
     │ 74HC154 /    │  ──── test point 0..255 ─│ CD74HC4067 ×16    │
     │ high-side sw │                          │ → comparator/ADC  │
     └──────────────┘                          └───────────────────┘
```

**Method:** drive one point at a time to 5 V through a series resistor (1–10 kΩ, current limited to <10 mA so nothing is stressed), scan all other points, record which read high. Result is a measured net-list. Compare against the expected net-list for the current step.

**Practical points that will bite you if ignored:**

- **Sneak paths through splices.** Automotive sub-harnesses are full of joints and splices; a single drive point can legitimately light up five sense points. Your comparison must be against the *expected* net set, not a naive 1:1 map.
- **Resistance threshold.** A terminal touching the pogo pin but not locked can still show continuity. Measure *resistance*, not just presence. Set a low threshold (a few ohms) so a marginal contact reads NG. Cirris-class instruments resolve down to milliohms; a 12-bit ADC with a known series resistor gets you to a usable few-ohm resolution cheaply.
- **Speed.** 256 × 256 with a 20 µs settle is ~1.3 s if done naively. Scan only the points relevant to the current step and you are under 20 ms. Full-board verification at the end of the harness can afford the full sweep.
- **ESD.** Every test point is a wire the operator touches. TVS diode array on every channel, ground the bench, wrist straps, and a series resistor before the mux.
- **Probes.** Spring-loaded pogo pins, 0.5–1.0 mm tip, 50–100 gf. Budget for replacement — these are the wear item, expect to swap them at a defined interval.

**Scale:** 256 points covers most sub-assy jobs. Build the drive/sense as a 32-channel module and stack eight of them. Expandable, and a failed module costs USD 20 not USD 500.

## 3.4 Cavity indication (adding Family A)

For 2.2 mm and 0.64 mm terminal cavities you **cannot** put a WS2812 behind each hole — it is physically too large. Two workable methods:

**Method 1 — Fibre light pipe (the CAMI approach).** 0.5–0.75 mm plastic optical fibre (PMMA), one strand per cavity, routed from a PCB of 0603 SMD LEDs to the back of each cavity. The terminal blocks the light when seated, so you get optical insertion confirmation as a bonus that is independent of the continuity check. Fibre is cheap; the labour is in the routing and potting.

**Method 2 — Side-fire SMD on a stacked PCB.** For connectors with ≥2.5 mm pitch, a PCB with 0402 side-view LEDs directly behind the cavity plane works and is cheaper to build. Below 2.5 mm pitch, go to fibre.

Drive the LEDs with a constant-current driver (TLC5947, 24 channels, or a chain of TLC59711) so brightness is stable and you can flash/pulse patterns: **solid = target cavity, fast flash = wrong insertion detected, all-flash = seal plugs required**.

## 3.5 Pull-force sensing (your TBO detector)

This is the part of your spec that most commercial systems *don't* do, and it is genuinely differentiating. Your defects board lists "Terminal Backout (TBO) / NP Lock" with a pull-check training panel — so the plant already does a manual pull check. Instrumenting it is high value.

**The naive approach:** load cell measures that the operator pulled hard enough. This only proves effort, not quality.

**The correct approach — measure force *and* continuity simultaneously:**

```
PASS  = peak force ≥ F_threshold  AND  continuity maintained throughout the pull
FAIL  = force reached threshold but continuity dropped   → terminal backed out (TBO)
FAIL  = force never reached threshold                    → operator under-pulled, retry
```

That is a true poka-yoke: it cannot be defeated by a half-hearted tug, and it directly detects the locking-lance failure you care about.

**Hardware:**
- **Load cell:** 5–10 kg straight-bar or S-type, mounted so the jig plate floats on it; or a smaller in-line cell on the wire-hook if you pull a single wire at a time.
- **ADC:** **NAU7802** (24-bit, I²C, up to 320 SPS) — much better than the ubiquitous HX711, whose 10/80 SPS is too slow to capture a peak cleanly. ADS1232 is the industrial-grade alternative.
- **Sampling:** ≥200 SPS, capture the full force-vs-time curve, log peak and the curve shape. Curve shape lets you spot a "slip then re-grip" that a peak-only reading would pass.
- **Calibration:** dead-weight calibration per station on a defined interval, stored in the jig node's EEPROM with a due date. Refuse to run past due. Automotive audits will ask for this.

**Setting F_threshold:** do *not* invent it. It must come from the connector/terminal supplier spec and be signed off by the customer's engineering. For orientation, the standards landscape: SAE/USCAR-21 governs crimp pull-out strength (pull rate 50–250 mm/min, 100 mm/min preferred); LV214 is the German OEM equivalent; JASO D616 covers Japanese automotive connector insertion/extraction performance. Terminal-in-cavity retention for small automotive terminals typically lands in the tens of newtons — but the number that goes in your config file is the customer's number, per part, not a textbook figure.

Store F_threshold **per connector part number per cavity** in the job program, not as a global constant.

## 3.6 Sensor coverage against your six defect modes

| # | Defect | Detection method | Confidence |
|---|---|---|---|
| 1 | Circuit wire miss | Continuity scan — expected net absent | High |
| 2 | Wrong wire insertion | Continuity scan — net at wrong probe | High |
| 3 | Retainer miss / open (TPA) | Micro-switch or inductive proximity sensor on the retainer travel in the jig; or a dedicated go/no-go plunger | Medium — needs per-connector mechanism |
| 4 | Terminal backout (TBO) | **Load cell + continuity-during-pull** (§3.5) | High |
| 5 | Accessory part missing (tube/tape/grommet) | Pick-to-light bins with IR beam-break at the bin mouth + step-gated confirmation; optionally a camera check | Medium — confirms a pick, not a placement |
| 6 | Male terminal pin bend | Machine vision (2 MP camera + ring light + template match) at a dedicated check pose, or a mechanical go/no-go comb gauge | Medium-high, but this is a separate sub-project |

**Honest scoping advice:** do #1, #2, #4 in v1. Add #3 and #5 in v2. Leave #6 (pin bend vision) out of the first machine entirely — it is a computer-vision project with its own lighting, dataset and false-reject tuning problem, and bolting it onto v1 will sink the schedule.

## 3.7 Peripherals and station furniture

| Item | Purpose | Suggested part |
|---|---|---|
| Touchscreen or monitor + keyboard | HMI (you already have Lenovo monitors — keep them) | 21.5" 1080p, or 15" capacitive touch |
| 2D barcode scanner | Scan job card 2488 to load the program | Honeywell/Zebra USB HID wedge |
| Foot pedal | Hands-free step advance | Industrial NO/NC pedal |
| Andon tower / beacon | You already have the red beacon — drive it from the controller | 3-colour 24 V tower + buzzer |
| NG / SB Start / Checker buttons | Keep the existing UI semantics; add physical illuminated buttons | 22 mm panel-mount illuminated |
| Wire cabin position lamps | "Wire Cabin: 0" on your screen — light the correct rack slot | Addressable LED strip along the rack, one per slot |
| Connector bin lights | Guide the correct connector bin (your blue bins) | Pick-to-light modules with IR beam-break |
| Jig ID reader | Auto-detect mounted fixture | 1-Wire / I²C EEPROM read over the RS-485 node |
| Power | Clean, isolated | 24 V DIN meanwell PSU + 5 V/3.3 V rails, EMI filter, surge arrestor |

## 3.8 Software architecture

```
┌─────────────────────────────────────────────────────────────┐
│  STATION PC                                                  │
│  ┌────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │ HMI        │  │ Job Program  │  │ MES / ERP adapter  │   │
│  │ (Electron  │  │ Compiler     │  │ (OPC-UA / REST /   │   │
│  │  or WPF or │  │ (Excel→JSON) │  │  file drop)        │   │
│  │  Qt)       │  └──────────────┘  └────────────────────┘   │
│  ├────────────┴──────────────────────────────────────────┐  │
│  │ Local store: SQLite (station) ⇄ PostgreSQL (plant)    │  │
│  └───────────────────────────────────────────────────────┘  │
│                    │ Ethernet / JSON-over-TCP or MQTT        │
└────────────────────┼─────────────────────────────────────────┘
                     ▼
┌─────────────────────────────────────────────────────────────┐
│  STATION CONTROLLER (ESP32-S3)                               │
│  step state machine · scan sequencer · interlocks · logging  │
└────────────────────┬─────────────────────────────────────────┘
                     │ RS-485 (Modbus RTU, 500 kbps)
        ┌────────────┼────────────┬────────────┐
        ▼            ▼            ▼            ▼
   Jig Node 1   Jig Node 2   Load-cell    Pick-to-light
   (32 ch)      (32 ch)      Node          Node
```

**Language choices:** HMI in Electron/TypeScript (fastest to build a good-looking, drawing-heavy UI, easy to render connector graphics as SVG) or C#/WPF if the plant is a Microsoft shop. Firmware in C with ESP-IDF (not Arduino — you want proper FreeRTOS task control, OTA rollback and partition tables).

**Critical rule:** the controller must be able to run a complete job from its own cached copy of the step file if the PC crashes mid-build. The PC is a display and a database, not a dependency.

---

# PART 4 — JOB PROGRAMS: EXCEL IN, GUIDANCE OUT

This is your "programs stored in the machine can be multiple, uploaded from an Excel sheet" requirement. Here is how to do it properly.

## 4.1 The key architectural decision

**Job programs must never be firmware.** They are versioned data files. A new harness = a new row set in a database. Firmware OTA is reserved for genuine firmware changes and happens maybe four times a year, signed and staged.

Concretely:
- Engineering uploads an Excel workbook → PC compiles it to a validated JSON step file → stored in the plant DB with a version and hash
- Operator scans job card 2488 → PC pushes the compiled step file to the controller over Ethernet → controller caches it in SPIFFS/LittleFS
- Controller executes. No flash cycle. Changeover in under two seconds.

## 4.2 Excel schema

Three sheets. Keep it flat and boring — engineers will edit this by hand.

**Sheet `HEADER`**
| Field | Example |
|---|---|
| harness_pn | YARIS-046d |
| variant | A |
| revision | 3 |
| description | Instrument panel sub-assy, RH |
| created_by / date | — |

**Sheet `CONNECTORS`**
| ref | connector_pn | cavities | jig_id | orientation | retainer_check | notes |
|---|---|---|---|---|---|---|
| C1 | ASZLE6-35F | 6 | JIG-0412 | A-up | TRUE | |
| C2 | SSGTSN8-18M-L | 18 | JIG-0876 | A-up | TRUE | |

**Sheet `CIRCUITS`** — one row per wire, this is the core table
| step | circuit_no | from_ref | from_cav | to_ref | to_cav | gauge | colour | length_mm | terminal_pn | seal_pn | pull_force_N | accessory | wire_cabin |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 33 | YQ4 | C1 | D | C2 | 7 | 0.35 | L | 1512 | 7116-xxxx | — | 30 | — | 0 |

Optional **Sheet `ACCESSORY`** for tubes, tapes, clips and their step positions.

Notice this maps 1:1 onto what your current screen already displays — page no, circuit no, gauge, colour, length, wire cabin, the two connector P/Ns and the highlighted cavity. That is deliberate; operators should see no change in information, only in reliability.

## 4.3 The compiler (the part that earns its keep)

The Excel→JSON compiler is where you prevent bad programs from ever reaching the floor. It must **reject**, not warn, on:

- A cavity referenced that doesn't exist in the declared connector
- Two wires assigned to the same cavity
- A connector P/N with no jig in the fixture library
- Missing pull_force_N where the connector requires a pull check
- Gauge/terminal mismatch against the terminal library
- Duplicate or non-contiguous step numbers
- Any circuit with only one end defined (unless explicitly flagged as a flying lead)

Output:

```json
{
  "harness_pn": "YARIS-046d",
  "revision": 3,
  "hash": "sha256:9f2c...",
  "compiled_utc": "2026-08-25T12:04:00Z",
  "connectors": [
    {"ref":"C1","pn":"ASZLE6-35F","jig":"JIG-0412","cavities":6},
    {"ref":"C2","pn":"SSGTSN8-18M-L","jig":"JIG-0876","cavities":18}
  ],
  "steps": [
    {
      "n": 33,
      "circuit": "YQ4",
      "wire": {"gauge":0.35,"colour":"L","length_mm":1512,"cabin":0},
      "insert": {"ref":"C1","cav":"D"},
      "expect_net": ["C1.D","C2.7"],
      "max_resistance_ohm": 2.0,
      "pull": {"force_N": 30, "hold_ms": 1000, "monitor_continuity": true}
    }
  ]
}
```

`expect_net` is the list of test points that must be electrically common after this step — this is what makes splices work correctly.

## 4.4 Distribution paths

| Path | Use case | Mechanism |
|---|---|---|
| **Network (primary)** | Normal operation | PC pulls from plant PostgreSQL, pushes compiled step file to controller over TCP. Controller validates hash before accepting. |
| **USB (fallback)** | Network down, or a standalone station | Signed `.job` bundle on a USB stick; PC verifies signature, imports, pushes. |
| **MQTT broadcast** | Fleet-wide program rollout | Publish new revision to `plant/harness/+/program`; stations subscribe and pre-cache. |
| **Firmware OTA (rare)** | Firmware only | `esp_ota` with dual partitions and automatic rollback; signed images; staged rollout (one station → one line → plant); never during a shift. |

**Security note that will matter at audit:** sign the compiled job files. An unsigned Excel file that anyone can edit and drop on a USB stick is a change-control finding waiting to happen. A simple Ed25519 signature from the engineering workstation's key, verified by the station PC, closes it.

## 4.5 Traceability output

Every completed unit writes a record:

```
job_card, harness_pn, revision, hash, station_id, operator_id,
unit_serial, start_utc, end_utc,
per_step: {step_n, insert_time_ms, resistance_mΩ, pull_peak_N, result},
ng_events: [{step_n, code, timestamp, cleared_by}]
```

This is what turns the machine from a productivity tool into a quality-system asset — and it is what lets you answer an OEM containment request with data rather than apology.

---

# PART 5 — BUILD STRATEGY

## Phase 0 — Gemba and specification (2–3 weeks)
Before any hardware. Sit at station A-2551 with a stopwatch and a notebook.

- Time-study the current cycle: seconds per insertion, per changeover, per NG event
- Count defect modes actually occurring, by frequency, over four weeks of checker data — this tells you which of the six to solve first, and it is your ROI baseline
- Inventory the connector population: how many distinct P/Ns across the running harnesses? Pareto them. If 20 connectors cover 80% of insertions, you need 20 jigs, not 150
- Pull the terminal retention specs for those 20 from the connector supplier
- **Confirm the change-control path.** This is a customer-approved process on an automotive line. Modifying or replacing a poka-yoke device will need customer engineering sign-off and probably a PPAP process-change submission. Find out now, not after you have a working machine.

**Deliverable:** a URS (user requirement spec) signed by production, quality and the customer's SQE.

## Phase 1 — Screen-only prototype (4–6 weeks)
Family C. No custom PCB yet.

- Mini-PC + monitor + barcode scanner + foot pedal + USB relay board for the beacon
- Full Excel→JSON compiler and HMI
- Runs a real job card end to end, guidance only, logging every step

**Why start here:** you validate the data model, the UI and the Excel workflow — the parts most likely to be wrong — before committing to hardware. And it is independently useful; a screen-only station is already better than paper.

## Phase 2 — Single-jig continuity node (6–8 weeks)
- Design and fab the 32-channel jig node PCB (RP2040 + mux + pogo pins + LED driver + EEPROM)
- Build one jig for your highest-volume connector
- Prove: wire-touch identification, cavity LED indication, insertion verification, wrong-cavity detection, sub-20 ms scan
- Iterate the pogo-pin mechanical design — this will take more revisions than you expect

**Gate:** demonstrate 500 consecutive insertions with zero false rejects and zero false accepts. If you can't hit that, do not proceed.

## Phase 3 — Pull-force and full station (6–8 weeks)
- Load-cell node with NAU7802, force-vs-time capture
- Continuity-during-pull logic
- Calibration routine and certificate generation
- Station controller PCB (ESP32-S3 + Ethernet + RS-485 master)
- Pick-to-light on the connector bins
- Andon integration

## Phase 4 — Pilot on one line (8–12 weeks)
- One station in production, running alongside the existing process for the first two weeks (shadow mode — machine logs but doesn't block)
- Compare machine NG calls against checker findings. Tune thresholds
- Then switch to blocking mode
- Measure against the Phase 0 baseline: defects, cycle time, rework
- Operator feedback loop — if the operators hate it, it will be defeated, and no amount of engineering fixes that

## Phase 5 — Fleet rollout and MES (ongoing)
- Standardise the jig build process; get jig cost down by batching
- Plant-wide program database, central fixture library, calibration due-date tracking
- MES/ERP integration
- Then, and only then, consider Phase 6: vision for pin-bend and accessory verification

**Realistic total to a proven pilot station: 6–9 months** with a small team (1 embedded engineer, 1 software engineer, 1 mechanical/fixture designer, part-time IE support). Anyone promising three months hasn't built the fixtures yet.

## 5.1 Indicative BOM — one station (prototype quantities, USD)

| Item | Qty | Unit | Total |
|---|---|---|---|
| Fanless N100 mini-PC, 8 GB/256 GB | 1 | 220 | 220 |
| 21.5" monitor (reuse existing) | 1 | 0 | 0 |
| Station controller PCB (ESP32-S3 + W5500 + RS-485 + I/O), assembled | 1 | 60 | 60 |
| Jig node PCB, 32 ch, assembled | 4 | 25 | 100 |
| Pogo pins + receptacles | 128 | 0.35 | 45 |
| Plastic optical fibre + SMD LEDs + TLC5947 drivers | — | — | 70 |
| Load cell 10 kg + NAU7802 board + mount | 1 | 45 | 45 |
| 2D barcode scanner | 1 | 60 | 60 |
| Foot pedal, illuminated buttons, andon tower + buzzer | 1 set | 90 | 90 |
| Pick-to-light modules (6 bins) | 6 | 15 | 90 |
| 24 V PSU, DIN rail, filters, enclosure, cabling | 1 set | 160 | 160 |
| Bench frame modification / jig plate | 1 | 200 | 200 |
| **Electronics + station subtotal** | | | **~1,140** |
| Connector-specific jig (machined body, fibre routing, wiring, programming) | per connector | 250–450 | — |

With a 20-jig starter library at ~USD 350 each, a first fully-capable station lands around **USD 8,000–9,000** — roughly at or below a single imported Cirris-class system, and you own the design, the spares chain and the ability to build station #2 for USD 1,140 plus jigs.

## 5.2 Risk register

| Risk | Severity | Mitigation |
|---|---|---|
| **Customer change-control refusal** | Critical | Engage the OEM SQE in Phase 0. Run shadow mode. Never remove the existing poka-yoke until the new one is approved. |
| **Jig library cost exceeds budget** | High | Pareto the connectors. Design a modular jig: common baseplate + swappable connector insert, so only the insert is per-P/N. |
| Pogo-pin wear / false NG | High | Defined replacement interval, gold-plated tips, daily self-test with a golden sample harness. |
| False rejects erode operator trust | High | Shadow mode first; tune thresholds on real data; give operators a fast, logged escalation path — never a silent override. |
| Load-cell drift | Medium | Scheduled dead-weight calibration, refuse-to-run past due, log every calibration. |
| ESD damage to scan channels | Medium | TVS on every channel, series resistors, bench grounding, wrist straps. |
| Wi-Fi unreliability | Medium | Wired Ethernet from day one. |
| Key-person dependency | Medium | Document the protocol and schema; keep the HAL portable; don't let the firmware live in one person's head. |
| **IP** | Medium | Do not reverse-engineer or clone the incumbent vendor's machine. Build from the published patent literature (much of which is expired) and your own design. Keep a clean-room record. |

---

# PART 6 — DECISION SUMMARY

**Architecture:** Family B (continuity-verified) backbone + Family A (cavity illumination) overlay, with Family C fallback. Family D (projected AR) deferred to the layout board as a separate project.

**Station PC:** fanless Intel N100 mini-PC, Windows 11 IoT LTSC or Ubuntu kiosk. Owns HMI, Excel compiler, database, MES link.

**Station controller:** ESP32-S3 + wired Ethernet on a custom PCB. STM32H5 as the production-hardening path.

**Jig nodes:** RP2040 or STM32G031, 32 channels each, RS-485/Modbus RTU daisy chain, each carrying an ID EEPROM so fixtures are self-identifying and hot-swappable.

**Scanning:** one-hot drive + 16:1 mux sense, resistance-thresholded, net-list comparison (not pin-to-pin), 256 points expandable in 32-channel modules.

**Cavity indication:** 0.75 mm plastic optical fibre from 0603 SMD LEDs, constant-current driven, terminal blocks the light as free optical confirmation.

**Pull check:** 10 kg load cell + NAU7802 at ≥200 SPS, pass requires peak force ≥ spec **and** continuity maintained throughout — this is your true TBO detector and your main differentiator.

**Programs:** Excel → validated, signed JSON step files in a plant database. Pushed over Ethernet, cached on the controller, USB and MQTT as alternate paths. **Job changes never touch firmware.** Firmware OTA is separate, signed, staged, dual-partition with rollback.

**First move:** Phase 0. Four weeks of checker data and a connector Pareto will tell you more about what this machine needs to be than any amount of further catalogue reading.

---

## Sources consulted

- CAMI Research, *Computer-Guided Harness Assembly* (C. E. Strangio) — Light Director architecture, first/second-sided pinning, published cost-benefit model
- Cirris Inc. — 8100 / CH2 / CR tester specifications, Easy-Wire guided assembly, Smart-Lights self-identifying adapters, harness board fixturing notes
- US Patent 6,993,833 (Sumitomo Wiring Systems) — *Apparatus for manufacturing sub-harness*: continuity-verified guided insertion loop
- US Patent 5,477,606 — *Assembly guiding apparatus for wiring harness subassemblies*: connector-receptor changeover problem
- US Patent 6,169,934 (Yazaki) — wire harness manufacturing system, FPS vs OES, sub-assy step definition
- US Patent 4,849,743 — terminal placement and degree-of-insertion determination
- SAE/USCAR-21 Rev. 3 — crimp pull-test methodology; LV214; JASO D616 — connector insertion/extraction performance
- Mecmesin / Imada — pull-test standards, rates and fixturing practice
- LightGuide, Arkite — projected-AR operator guidance for wire harness and cabling
- ASSEMBLY Magazine — historical survey of harness automation attempts
- DirectIndustry, TradeIndia, Made-in-China supplier listings — regional SPM and insertion machine market
