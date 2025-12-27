# Emergency Call Box Optimization at McGill Downtown Campus
<img width="5972" height="2956" alt="emergency_callbox_optimization" src="https://github.com/user-attachments/assets/f22b697b-c82e-4726-956f-8876bfa389dc" />

---
## Problem Overview

McGill’s emergency call box network has gaps: some call boxes are outdated, others are in poor condition and overall coverage is insufficient. This project formulates a **facility location model** to decide which existing call boxes to keep or replace, where to install new call boxes, and which phone type to deploy at each location, while **meeting a minimum coverage threshold** at the **lowest total cost**.

---
## Repository Structure
```
├── data/
│   ├── campus_map.pdf                          # Source map (existing call box locations, buildings, and routes)
│   ├── building_outline_vertices.csv           # Traced campus building boundaries
│   ├── solid_line_vertices.csv                 # Main night route vertices
│   ├── dotted_line_vertices.csv                # Feeder route vertices
│   ├── exisiting_and_candidate_callbox_coords.csv   # All 123 phone locations
│   ├── demand_point_coords.csv                 # 98 emergency phone demand points with weights
│   └── existing_callbox_current_state.csv      # Phone types & conditions at existing locations (Field Observations)
│
├── notebooks/
│   ├── pdf_map_coord_extraction.ipynb          # Step 1: Data extraction
│   └── callbox_optimization_gurobi.ipynb       # Step 2: Optimization model
│
└── README.md
```
---
## Notebooks

### 1. `pdf_map_coord_extraction.ipynb`

Extracts all spatial inputs from the campus PDF map:

- Renders PDF to high-res image (**PyMuPDF**) and detects existing call boxes via **HSV color filtering** + **contour detection**
- Builds an interactive **OpenCV** interface (`cv2.setMouseCallback`) to trace building outlines and route polylines
- Generates 100 candidate call box locations by sampling along building boundaries
- Samples emergency phone demand points at 50m intervals along routes using a pixel-to-meter scale calibrated from reference points, and applies 1.5× weights to main routes versus feeder routes

**Outputs:** All CSV files in data/

**Note:** Requires interactive input (clicking on the map). If you just want to run the optimization, the CSV files are already included

---
### 2. `callbox_optimization_gurobi.ipynb`

Solves the facility location problem using Gurobi:

- Builds **Euclidean distance matrix** from location/demand CSVs
- Binary decision variables: phone placement (`X`), coverage assignment (`A`), overall coverage indicator (`Y`)
- Minimizes installation costs + condition-based penalties for retaining old phones
- Fixes initial phone types, links coverage via type-specific radii with a minimum 65% weighted coverage target and restricts replacement at existing locations
- Outputs optimal decisions with before/after visualization

---
## Results

The model achieves **61.2% demand point coverage** and **65.1% weighted coverage** at a total cost of **$251,500 CAD**, installing 15 new phones and replacing 6 existing phones. Detailed results are documented in the `callbox_optimization_gurobi.ipynb`.

---
## Team

Rui Zhao, Serena Sun, Arantzazu Arregui Gonzalez, Chloee Liew, Nicholas Stanfield
