# Emergency Call Box Optimization at McGill Downtown Campus

## Problem Overview

McGill's existing emergency phone network has gaps: some phones are outdated, others are in poor condition, and overall coverage is insufficient. This project formulates a **facility location problem** that decides which existing phones to upgrade or keep, where to install new phones and what phone type to use at each location while meeting **a minimum coverage threshold** at **minimum cost**.

## Repository Structure

├── data/
│   ├── campus_map.pdf                          # Source map
│   ├── building_outline_vertices.csv           # Traced campus building boundaries
│   ├── solid_line_vertices.csv                 # Main night route vertices
│   ├── dotted_line_vertices.csv                # Feeder route vertices
│   ├── exisiting_and_candidate_callbox_coords.csv   # All 123 phone locations
│   ├── demand_point_coords.csv                 # 98 demand points with weights
│   └── existing_callbox_current_state.csv      # Phone types & conditions at existing call boxes
│
├── notebooks/
│   ├── pdf_map_coord_extraction.ipynb          # Step 1: Data extraction
│   └── callbox_optimization_gurobi.ipynb       # Step 2: Optimization model
│
└── README.md
```

---

## Notebooks

### 1. `pdf_map_coord_extraction.ipynb` — Data Pipeline

Extracts all spatial inputs from the campus PDF map.

**What it does:**
- Renders the PDF to a high-resolution image using PyMuPDF
- Detects existing call box locations via HSV color filtering + contour detection
- Provides an interactive OpenCV interface to manually trace building outlines and route polylines
- Generates 100 candidate locations by sampling points along building perimeters
- Samples demand points at 50m intervals along traced routes (using a pixel-to-meter scale derived from known reference points)
- Assigns higher weights (1.5×) to main routes vs. feeder routes

**Key techniques:** PDF rendering, color-space segmentation, interactive annotation with `cv2.setMouseCallback`, polyline interpolation, coordinate system calibration.

**Outputs:** All CSV files in `data/`

---

### 2. `callbox_optimization_gurobi.ipynb` — MIP Model

Solves the facility location problem using Gurobi.

**What it does:**
- Loads location and demand data, builds a Euclidean distance matrix
- Defines binary decision variables: phone placement (`X`), demand-to-phone assignment (`A`), coverage indicator (`Y`)
- Minimizes: equipment costs + condition-based penalties for retaining old phones
- Enforces: coverage radius limits, minimum weighted coverage (65%), no-removal and no-downgrade constraints for existing sites
- Outputs optimal decisions and generates a side-by-side comparison plot

**Key constraints:**
- Each demand point can only be served by a phone within its coverage radius
- Existing locations must retain *some* phone (no removal)
- Modern phones cannot be downgraded to older types
- At least 65% weighted coverage required

**Key techniques:** Mixed-integer programming, set covering formulation, multi-period state modeling, Gurobi API.

**Outputs:** `emergency_callbox_optimization.png`, solver logs, decision summary

---

## Execution Order

Run the notebooks in sequence—each step depends on outputs from the previous:

| Step | Notebook | Inputs | Outputs |
|------|----------|--------|---------|
| 1 | `pdf_map_coord_extraction.ipynb` | `campus_map.pdf` | All CSVs in `data/` |
| 2 | `callbox_optimization_gurobi.ipynb` | CSVs from step 1 | Optimal solution + visualization |

**Note:** Step 1 requires interactive input (clicking on the map). If you just want to run the optimization, the CSV files are already included.

---

## Requirements

```
gurobipy          # Requires a Gurobi license (free academic license available)
pandas
numpy
matplotlib
seaborn
opencv-python
PyMuPDF           # fitz
Pillow
```

---

## Results

The model achieves **65.3% demand coverage** at a total cost of **$251,500**, installing 19 new phones and upgrading 4 existing locations. Full results and sensitivity analysis are documented in the project report.

---

## Team

Arantzazu Arregui Gonzalez · Chloee Liew · Nicholas Stanfield · Rui Zhao · Serena Sun
