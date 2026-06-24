# Post-War Building Assessment — Fuzzy Expert System

A fuzzy logic expert system built with **Python** and **scikit-fuzzy** that assesses war-damaged buildings, estimates damage severity, and provides actionable reconstruction recommendations — all while handling incomplete or uncertain input data.

---

## Project Overview

Post-war building assessment faces three fundamental problems:

- **Inaccurate data** — dangerous access conditions prevent thorough inspection
- **Incomplete data** — lost blueprints, displaced owners, destroyed records
- **Conflicting data** — shortage of qualified engineers and unreliable self-assessments

This system addresses all three using **fuzzy logic**: inputs can be linguistic labels (`"Major Crack"`, `"Unknown"`, `"High"`) or normalized numeric values, and the system infers outputs even when data is missing or uncertain.

---

## Features

- **5 fuzzy inputs** covering structural safety, damage type, repair economics, and site risk
- **5 fuzzy outputs** — structural factor, partial damage level, total damage, risk level, and recommendation
- **35 IF-THEN fuzzy rules** encoding expert engineering knowledge
- **Uncertainty handling** — `"Unknown"` values trigger fallback rules instead of failing
- **Dual input mode** — linguistic labels (strings) or numeric values (0–1 range)
- **Fuzzy set visualization** — membership function plots for all output variables
- **4 built-in test cases** — auto-run when the user provides invalid input

---

## System Architecture

### Inputs

| Variable | Type | Range / Labels |
|---|---|---|
| `structural_safety` | Discrete | Unknown, Primary Column, Secondary Column, Foundation, Ceiling, Wall, Floor, Staircase, Water Pipe, Sewage Pipe, Electrical Network, Door, Window, Fence |
| `damage_type` | Discrete | Unknown, Major Crack, Partial Collapse, Significant Tilt, Major Leak, Severe Corrosion, Heavy Rust, Minor Crack, Minor Leak, Mild Corrosion, Light Rust, Partial Settlement, Mild Tilt, No Damage |
| `repair_cost` | Continuous | 0.0–1.0 → Very Low / Low / Medium / High / Very High / Unknown |
| `repair_time` | Continuous | 0.0–1.0 → Very Short / Short / Medium / Long / Very Long / Unknown |
| `risk_factor` | Discrete | Unknown, Unexploded Ordnance, Large Hanging Debris, Active Fire, Imminent Collapse, Inactive Ordnance, Small Hanging Debris, Stable Fire Residues, Uncertain Collapse, No Risk Factors |

### Intermediate Outputs

| Variable | Labels |
|---|---|
| `structural_factor` | Main / Secondary / Supplementary / Unknown |
| `partial_damage_level` | Deep / Surface / Nothing / Unknown |

### Final Outputs

| Variable | Labels |
|---|---|
| `total_damage` | Severe / Moderate / Minor / Safe / Unknown |
| `risk_level` | High / Medium / Low / Safe / Unknown |
| `recommendation` | 10 distinct action recommendations (see Rules section) |

---

## Fuzzy Inference Pipeline

```
5 Inputs (linguistic or numeric)
    │
    ▼
Fuzzification
(Trimf membership functions map inputs to fuzzy degrees)
    │
    ▼
Rule Evaluation (35 IF-THEN rules in 7 groups)
    │
    ├─ Group 1: structural_safety     → structural_factor
    ├─ Group 2: damage_type           → partial_damage_level
    ├─ Group 3: structural_factor
    │           + partial_damage_level → total_damage
    ├─ Group 4: repair_cost
    │           + repair_time
    │           + partial_damage_level → total_damage
    ├─ Group 5: risk_factor            → risk_level
    ├─ Group 6: structural_factor
    │           + partial_damage_level → risk_level
    └─ Group 7: total_damage
                + risk_level           → recommendation
    │
    ▼
Defuzzification (centroid method via skfuzzy)
    │
    ▼
Outputs:
  - Total Damage Level (ratio 0–1 + label)
  - Recommendation (ratio 0–1 + label)
  - Membership function plots
```

---

## Fuzzy Rules Summary

**Structural Safety → Structural Factor**
- Primary Column / Secondary Column / Foundation / Ceiling → **Main**
- Wall / Floor / Staircase / Water Pipe / Sewage Pipe / Electrical Network → **Secondary**
- Door / Window / Fence → **Supplementary**

**Damage Type → Partial Damage Level**
- Major Crack / Partial Collapse / Significant Tilt / Major Leak / Severe Corrosion / Heavy Rust → **Deep**
- Minor Crack / Minor Leak / Mild Corrosion / Light Rust / Partial Settlement / Mild Tilt → **Surface**
- No Damage → **Nothing**

**Total Damage (sample rules)**
- Main + Deep → **Severe**
- Main + Surface → **Moderate**
- Secondary + Deep → **Moderate**
- Supplementary + Surface/Nothing → **Safe**
- Very High Cost + Very Long Time → **Severe**

**Recommendations**
| Total Damage | Risk Level | Recommendation |
|---|---|---|
| Severe | High | Immediate evacuation and no entry, followed by demolition and reconstruction |
| Severe | Medium | Temporary evacuation and securing the site, followed by demolition and reconstruction |
| Moderate | High | Evacuate affected areas, monitor using drones, and perform urgent repairs |
| Moderate | Medium | Secure and monitor the site using drones, and perform repairs as soon as possible |
| Moderate | Safe | Comprehensive repair before use |
| Minor | Safe | Perform routine maintenance and minor improvements |
| Minor | High | Perform minor repairs to prevent increased risk |
| Safe | Safe | Regular use without restrictions |

---

## Requirements

```bash
pip install scikit-fuzzy numpy matplotlib
```

---

## Usage

```bash
jupyter notebook post_war_building_assessments.ipynb
```

Or run as a script by executing the final cell. On launch, the system prompts:

```
Do you want to enter a string or numeric values?
(enter 1 for string, 2 for numeric)
```

**Option 1 — Linguistic input:**
```
Enter structural safety: Primary Column
Enter damage type: Major Leak
Enter repair cost: Very High
Enter repair time: Very Long
Enter risk factor: Unexploded Ordnance
```

**Option 2 — Numeric input:**
```
Enter structural safety (0 to 12): 1
Enter damage type (0 to 12): 4
Enter repair cost (0 to 1): 0.9
Enter repair time (0 to 1): 0.9
Enter risk factor (0 to 8): 1
```

**Sample output:**
```
Simulation results...
Total Damage Level (Ratio): 0.85
Recommendation (Ratio): 0.97
Total Damage Level: Severe
Recommendation: Immediate evacuation and no entry, followed by demolition and reconstruction
```

If an invalid value is entered, the system automatically runs 4 built-in test cases.

---

## Test Cases

| # | Scenario | Structural Safety | Damage Type | Cost | Time | Risk Factor | Result |
|---|---|---|---|---|---|---|---|
| 1 | Safe | Door | No Damage | Very Low | Very Short | No Risk Factors | **Safe (10%)** |
| 2 | Minor | Wall | Mild Corrosion | Low | Short | No Risk Factors | **Minor (24%)** |
| 3 | Moderate | Staircase | Minor Crack | Medium | Long | Inactive Ordnance | **Moderate (50%)** |
| 4 | Severe | Primary Column | Major Leak | Unknown | Very Long | Unexploded Ordnance | **Severe (85%)** |

---

## Project Structure

```
post-war-building-assessment/
│
├── post_war_building_assessments.ipynb   # Main Jupyter notebook (full pipeline)
│
├── report/
│   └── Post_war_building_assessments.pdf # Full project report
│
└── README.md
```

---

## Key Functions

| Function | Description |
|---|---|
| `define_variables()` | Defines all 10 fuzzy linguistic variables with trimf membership functions |
| `get_output_label()` | Maps a defuzzified numeric output back to its linguistic label |
| `get_label()` | Wrapper to extract the label for any output variable from simulation results |
| `construct_rules()` | Builds all 35 fuzzy IF-THEN rules with uncertainty fallback conditions |
| `create_fuzzy_system()` | Assembles rules into a `ControlSystem` and returns a `ControlSystemSimulation` |
| `evaluate_system()` | Handles user input (string/numeric), runs inference, prints results |
| `visualize_system()` | Plots membership functions for all output variables using skfuzzy's `.view()` |

---

## Libraries Used

| Library | Purpose |
|---|---|
| `scikit-fuzzy` | Fuzzy sets, membership functions, control system, simulation |
| `numpy` | Universe arrays and numerical operations |
| `matplotlib` | Visualizing fuzzy membership function plots |

---

## Future Improvements

- Integrate machine learning to auto-generate rules from historical assessment data
- Replace manual trimf membership functions with data-driven shapes (Gaussian, trapezoidal)
- Use TensorFlow Fuzzy for faster inference on large-scale datasets
- Build a web or mobile interface for field engineers to use on-site
- Export assessment reports as structured PDF documents
