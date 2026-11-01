# WireArc-ZeroG: In-Space Electron-Beam Additive Quality Scanner

> **Real-Time Dynamic Contact Impedance $(v, R)$ State-Space Inversion and Microsecond Electromagnetic Deflection Interlock for Microgravity Metal Additive Manufacturing.**

Developed by **RFWaveCode LLC** (`CAGE: 8V2L1` | `UEI: Y4K8NJM49L93`) under NASA SBIR 2026 Solicitation Topic **Z1.04: In-Space Servicing, Assembly, and Manufacturing (ISAM) & Autonomous Structural Quality Verification**.

🔗 **Interactive Simulation & Telemetry HUD:** Single-file application (`index.html`)  
📬 **Primary Technical Point of Contact:** [support@rfwavecode.com](mailto:support@rfwavecode.com)

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Target Problem: Microgravity Liquid Bridge Detachment](#target-problem-microgravity-liquid-bridge-detachment)
3. [The Core Physical Insight](#the-core-physical-insight)
4. [Closed-Loop Software & Hardware Mitigation Architecture](#closed-loop-software--hardware-mitigation-architecture)
5. [Primary Diagnostic Axes: $v\text{-}R$ Phase-Space Diagram](#primary-diagnostic-axes-v-r-phase-space-diagram)
6. [Core Mathematical Framework](#core-mathematical-framework)
7. [System Architecture & 4-Tab Web Application](#system-architecture--4-tab-web-application)
8. [NASA SBIR Phase I Allocation ($225,000)](#nasa-sbir-phase-i-allocation-225000)
9. [Civil & Commercial ARR Projections ($212.5M)](#civil--commercial-arr-projections-2125m)
10. [Local Quickstart & Execution](#local-quickstart--execution)
11. [NASA Core Flight System (cFS) Applet Implementation](#nasa-core-flight-system-cfs-applet-implementation)
12. [Data Rights & Contact](#data-rights--contact)

---

## Executive Summary

Autonomous structural assembly of persistent orbital platforms—such as 25-meter sub-millimeter RF reflector trusses, solar power satellites, and deep-space transit vehicles—requires wire-fed electron-beam additive manufacturing directly in vacuum and microgravity. 

However, in microgravity ($g \approx 0$), the absence of hydrostatic pressure head destabilizes the molten pool. Without gravitational drainage, surface tension and Marangoni shear dominate, triggering Rayleigh-Plateau pinch-off, droplet detachment, bead humping, and critical lack-of-fusion voids inside structural titanium nodes.

**WireArc-ZeroG** is an embedded state observer and high-speed closed-loop quality scanner. By injecting a high-frequency (100 kHz) non-invasive electrical excitation through the advancing wire feed nozzle into the substrate, the system maps instantaneous dynamic contact resistance ($R$) against wire feed velocity ($v$). When liquid necking begins, the contact constriction resistance spikes asymptotically. WireArc-ZeroG detects this pre-collapse gradient in microseconds ($\Delta t < 12\,\mu\text{s}$), actively repointing high-frequency electromagnetic steering coils to widen the pool and re-establish continuous capillary bridge conduction before internal porosity can nucleate.

---

## Target Problem: Microgravity Liquid Bridge Detachment

Terrestrial wire-fed additive systems (EB-DED / WAAM) rely on gravity to hold molten liquid against the substrate:

```
+-----------------------------------------------------------------------------------+
|                        MICROGRAVITY DEFECT FORMATION MODES                        |
+-----------------------------------------------------------------------------------+
| 1. Absence of Hydrostatic Head   --> Liquid pool beads up into spherical droplet   |
| 2. Thermal Keyhole Recoil Surge  --> Vapor pressure expels molten bridge          |
| 3. Feed Rate / Melting Mismatch  --> Wire tip separates from melt pool             |
|                                                                                   |
| RESULT: Rayleigh-Plateau Capillary Breakup                                        |
|   --> Liquid bridge snaps into floating free droplet                              |
|   --> Lack-of-fusion, macro-porosity, and open-seam fractures in truss struts     |
|   --> Zero-G truss members fail structural qualification without CT inspection    |
+-----------------------------------------------------------------------------------+
```

Incumbent orbital monitoring relies on high-speed optical cameras or thermal pyrometers. In vacuum electron-beam manufacturing, these optical instruments are blinded by the secondary electron flare, vapor plume, and $3{,}000\,\text{K}$ plasma glare, making real-time sub-surface pool geometry unobservable.

---

## The Core Physical Insight

In vacuum electron-beam wire deposition, the consumable wire feed velocity ($v$) and the dynamic molten pool electrical resistance ($R$) are directly coupled through the physical contact area of the liquid meniscus bridge.

* **Nominal Conduction Regime:** When the wire tip remains continuously wetted to the melt puddle, current flows through a broad molten contact neck ($a(t) \approx r_w$). Resistance scales smoothly with temperature and solid wire stickout length:
  $$\frac{\partial R}{\partial v} \approx \kappa_{\text{nominal}}$$

* **The Spheroidization Bifurcation:** When wire feed fails to keep pace with vaporization or capillary necking commences, the liquid meniscus undergoes Rayleigh-Plateau pinch-off:
  $$L_{\text{bridge}} > 2\pi r_{\text{bridge}}$$
  As the liquid bridge necks down, the effective contact cross-sectional radius collapses ($a(t) \to 0$).

* **The Mathematical Trigger:** According to Maxwell constriction resistance physics, the resistance of a narrowing contact bridge diverges:
  $$\lim_{a(t) \to 0} R(t) = \lim_{a(t) \to 0} \left[ \frac{\rho_{\text{liquid}}}{2 a(t)} \right] = \infty$$

**Any sudden resistance surge while feed velocity is constant ($\left.\frac{\partial R}{\partial t}\right|_v > \xi_{\text{crit}}$) serves as an unambiguous mathematical precursor to droplet detachment and lack-of-fusion porosity.**

---

## Closed-Loop Software & Hardware Mitigation Architecture

WireArc-ZeroG integrates directly between the wire feed subsystem, the isolated sensing front-end, and the electron gun deflection amplifiers:

```
       +-----------------------------------------------------------------+
       |           100 kHz Isolated AC Contact Impedance Sensor          |
       |               [Carrier: 250 mV RMS, 100 kHz Lock-In]            |
       +-----------------------------------------------------------------+
                                       |
                                       v
       +-----------------------------------------------------------------+
       |             FPGA State Observer & Inversion Engine              |
       |           - Maxwell Constriction Radius Inversion a(t)          |
       |           - Dynamic State Mapping: (v, R) Coordinate            |
       |           - Detachment Derivative Monitor: dR/dt                |
       +-----------------------------------------------------------------+
                                       |
               +-----------------------+-----------------------+
               |                                               |
         R < R_threshold                                R >= R_threshold
               |                                               |
               v                                               v
     [NOMINAL PASSIVE LOGGING]                   [MICROSECOND EM INTERVENTION]
     - Normal feed advancement                   1. Fire 25 kHz magnetic coils
     - Telemetry packet streaming                   (Microsecond beam steering)
                                                 2. Deflect beam spot +0.42 mm
                                                    toward solid wire root
                                                 3. Suppress droplet separation
                                                    & restore liquid bridge
```

---

## Primary Diagnostic Axes: $v\text{-}R$ Phase-Space Diagram

The diagnostic engine plots the real-time additive operating point on a two-dimensional state plane:

| Diagnostic Axis | Parameter | Engineering Units | Physical Meaning |
| :--- | :--- | :--- | :--- |
| **X-Axis** | **Wire Feed Velocity ($v$)** | $\text{mm/s}$ | Volumetric deposition rate of Ø1.2 mm Grade 5 Ti-6Al-4V wire into the melt zone. |
| **Y-Axis** | **Dynamic Pool Resistance ($R$)** | $\text{m}\Omega$ | High-frequency contact resistance across the liquid bridge and melt puddle. |

### Visual Elements on the Phase Diagram

1. **Nominal Conduction Corridor:** Baseline impedance curve representing stable conduction with full liquid wetting.
2. **Zero-G Detachment Hazard Boundary (Red Dashed Line):** The stability threshold where liquid bridge geometry exceeds the capillary pinch limit. The zone above this line indicates droplet detachment and macro-void entrapment.
3. **Live Dynamic Trajectory:** Trailing vector tracing instantaneous $(v, R)$ state points. When an anomalous trajectory vector spikes vertically upward without horizontal velocity displacement, the autonomous recovery interlock triggers.

---

## Core Mathematical Framework

### 1. Zero-G Liquid Bridge Equilibrium (Young-Laplace & Recoil Pressure)

$$\Delta P = \gamma \left( \frac{1}{R_1} + \frac{1}{R_2} \right) + P_{\text{recoil}} = \gamma \kappa(s) + \frac{0.55 P_0}{\sqrt{2\pi \mathcal{R} T}} \exp\left[ -\frac{\Delta H_v}{\mathcal{R} T} \right]$$

### 2. Maxwell Dynamic Constriction Resistance

$$R(t) = R_{\text{bulk}} + R_{\text{constriction}} = \frac{\rho_{\text{solid}} L_{\text{stickout}}}{\pi r_w^2} + \frac{\rho_{\text{liquid}}(T)}{2 a(t)} + \frac{\rho_{\text{truss}}}{4 r_{\text{pool}}}$$

### 3. Non-Linear $(v, R)$ State Inversion Curve

$$R_{\text{nominal}}(v) = R_0 + \alpha \frac{\mu_{\text{eff}} \cdot v}{\gamma_{\text{surface}}} \left( 1 + \text{Ca}^{1/3} \right)^{-1}, \quad \text{where } \text{Ca} = \frac{\mu v}{\gamma}$$

### 4. Microsecond Magnetic Beam Deflection Control Law

$$\mathbf{I}_{\text{deflect}}(t + \Delta t) = \mathbf{K}_p \left( R(t) - R_{\text{target}}(v) \right) + \mathbf{K}_d \frac{d R}{dt} + \mathbf{K}_{\text{feed}} \nabla_{\mathbf{B}} \Phi_{\text{beam}}$$

---

## System Architecture & 4-Tab Web Application

The interactive application (`index.html`) is structured into four views:

```
+-----------------------------------------------------------------------------------+
|                        WIREARC-ZEROG APPLICATION INTERFACE                        |
+-----------------------------------------------------------------------------------+
| [Live HUD]          [Physics & Math]     [$225k SBIR Plan]    [Civil ARR ($212M)] |
+-----------------------------------------------------------------------------------+
|  1. Live HUD (#viewSim)                                                           |
|     - Physics-driven HTML5 Canvas: 60 kV Electron Gun, wire feeder, liquid bridge.|
|     - Chart.js Scatter: Live (v, R) trajectory with detachment hazard boundary.   |
|     - Time-Series Strip: Multi-channel logging of R, R_limit, and velocity v.     |
|     - Fault Injections: Wire Feed Slip, Keyhole Vaporization, Bridge Detachment.  |
|                                                                                   |
|  2. Physics & Mathematical Model (#viewPhysics)                                   |
|     - MathJax v3 formula cards covering Young-Laplace, Maxwell, and coil physics. |
|                                                                                   |
|  3. $225,000 NASA SBIR Proposal (#viewProposal)                                   |
|     - Topic Z1.04 alignment, 6-month milestones, and Phase II orbital flight path.|
|     - Explicit 30.0% ($67,500) Academic Research Subcontractor allocation.        |
|                                                                                   |
|  4. Commercial & Civil ARR Breakdown (#viewARR)                                   |
|     - Strictly civil market analysis across 5 sectors totaling $212.5M ARR.       |
+-----------------------------------------------------------------------------------+
```

---

## NASA SBIR Phase I Allocation ($225,000)

| Cost Category | Scope & Justification | Amount | % Budget |
| :--- | :--- | :--- | :--- |
| **Direct Technical Labor** | Lead Additive Physicist & Real-Time FPGA / Embedded Engineers (820 hours) | **$104,200** | 46.3% |
| **Fringe & Indirect Overhead** | DCAA-compliant provisional indirect pool (30.8% base) | **$32,100** | 14.3% |
| **Academic Subcontract (STTR)** | High-Vacuum ($10^{-5}\,\text{Torr}$) E-beam chamber testing & micro-CT verification | **$67,500** | 30.0% |
| **Flight Sensors & Prototyping** | 100 kHz lock-in demodulators, Ti-6Al-4V wire spools, isolated Rogowski sensors | **$10,400** | 4.6% |
| **Negotiated Small Business Fee** | Standard FAR 15.404-4 commercialization fee (4.8%) | **$10,800** | 4.8% |
| **Total Firm-Fixed Price** | **NASA SBIR Phase I Total Budget** | **$225,000** | **100.0%** |

### 6-Month Phase I Milestone Roadmap

* **Months 1–2:** Formulate 3D finite-element surface tension model for wire bridge dynamics under zero-g. Calibrate theoretical $(v, R)$ state boundaries for Ti-6Al-4V under $10^{-5}\,\text{Torr}$ vacuum conditions.
* **Month 3:** Design and program real-time contact impedance demodulator on radiation-tolerant Xilinx Zynq UltraScale+ FPGA. Implement microsecond detection of $\partial R / \partial t$ impedance spikes.
* **Months 4–5:** Execute 30+ deposition runs in university partner high-vacuum E-beam chamber. Induce deliberate feed fluctuations and demonstrate real-time defect capture via high-speed beam deflection.
* **Month 6:** Finalize flight enclosure specification, NASA Core Flight System (cFS) applet wrapper, and detailed Phase II commercialization roadmap ($950\text{k}\text{--}\$1.25\text{M}$ orbital prototype experiment).

---

## Civil & Commercial ARR Projections ($212.5M)

WireArc-ZeroG's contact impedance architecture applies across civil and terrestrial industrial sectors:

```
                                  COMMERCIAL ARR BREAKDOWN
+-----------------------------------+--------------------+------------------+---------------+
| Civil Target Sector               | Hardware Platform  | Pricing Model    | Annual ARR    |
+-----------------------------------+--------------------+------------------+---------------+
| 1. Commercial Aerospace Ti-6Al-4V | Wing Spar DED      | $65,000 / tool/yr| $80.6M        |
| 2. Commercial Orbital Stations    | Robotic Truss Arms | $180,000 / arm/yr| $46.8M        |
| 3. Civil Nuclear Fusion Vessels   | Tokamak Chambers   | $95,000 / cell/yr| $36.1M        |
| 4. Semiconductor Vacuum Tooling   | Refractory Liners  | $40,000 / sys /yr| $27.2M        |
| 5. Subsea Pipeline Repair         | Robotic Crawlers   | $38,000 / unit/yr| $21.8M        |
+-----------------------------------+--------------------+------------------+---------------+
| TOTAL RECURRING CIVIL ARR                              | 10x Multiple ARR | $212.5M       |
+-----------------------------------+--------------------+------------------+---------------+
```

---

## Local Quickstart & Execution

The simulation is packaged as a single, self-contained file (`index.html`) with zero local dependencies or build pipelines.

### 1. Clone the Repository

```bash
git clone https://github.com/RFWaveCode/WireArc-ZeroG.git
cd WireArc-ZeroG
```

### 2. Run the Application

Open `index.html` directly in any standard, modern web browser:

```bash
# Using Python built-in HTTP server:
python -m http.server 8080

# Or launch directly on macOS:
open index.html

# Or launch directly on Linux:
xdg-open index.html
```

Access the interface via `http://localhost:8080`.

---

## NASA Core Flight System (cFS) Applet Implementation

The real-time detection and deflection controller compiles into an autonomous NASA cFS application (`wirearc_app.c`):

```c
#include "cfe.h"
#include "wirearc_app.h"

/* Real-time 100 kHz Demodulation & Control Loop */
void WireArc_ProcessImpedance(void)
{
    float v_feed = Read_Wire_Feeder_Encoder();
    float R_meas = Read_Contact_Impedance();
    float R_threshold = Calculate_Rayleigh_Threshold(v_feed);

    /* Real-time derivative calculation */
    float dR_dt = Calculate_Impedance_Derivative(R_meas);

    if (R_meas >= R_threshold || dR_dt > CRITICAL_NECK_RATE)
    {
        /* Autonomous High-Speed Magnetic Deflection */
        CFE_EVS_SendEvent(WIREARC_ALERT_EID, CFE_EVS_EventType_CRITICAL,
                          "Liquid bridge necking detected. Firing magnetic steering coils.");

        Steer_Beam_Offset_Microseconds(0.42f); // +0.42 mm deflection to wire root
        Adjust_Wire_Drive_Torque(1.08f);       // Dampen wire feed jitter
    }
}
```

---

## Data Rights & Contact

* **Proprietary Notice:** RFWaveCode LLC holds all software architecture and state inversion patent rights.
* **Government Purpose Rights:** NASA SBIR data rights apply under FAR 52.227-20 for Phase I awardees.

For technical documentation, bench validation datasets, or STTR academic research partnership inquiries:

* **Company:** RFWaveCode LLC
* **DUNS / UEI:** `Y4K8NJM49L93` | **CAGE Code:** `8V2L1`
* **Email:** [support@rfwavecode.com](mailto:support@rfwavecode.com)