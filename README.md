# LunaP: Autonomous Hazard-Aware Lunar Surface Navigator

> **LunaP uses AI to understand lunar terrain, map hazards, rank landing sites, and plan safer routes.**

## Overview

LunaP is an autonomous, hazard-aware lunar navigation system that converts lunar optical imagery into terrain intelligence. Its AI/ML perception layer extracts terrain features and its pictures from publicly available database of lunar imagery. 

For the hackathon, the pipeline is modular and can combine annotations, pretrained visual representations, automated terrain indicators, and lightweight learning methods instead of exhaustive manual labeling or reviewing. For each image that has been uplaoded, LunaP produces a terrain-risk representation distinguishing safer and hazardous regions. 

A mission-planning layer uses this representation to generate routes between user-defined nodes and annotations while considering terrain risk and safety rather than distance alone. Candidate landing or staging regions are evaluated and ranked with the least risk and the safest. An interactive dashboard presents the image, terrain analysis, risk map, recommended route, landing-site ranking, and risk information. The prototype demonstrates an end-to-end workflow from terrain understanding to risk-aware planning and presentation.

## Problem Statement

Lunar terrain contains craters, rocks, steep or rough regions, shadows, and other features and risk factors that are visually difficult to view. A rover cannot safely treat every visible region as equally traversable. Communication delay also limits dependence on continuous human intervention. 

**LunaP addresses the problem of converting local lunar imagery into terrain-risk information that can support safer navigation and mission planning.**

## Proposed Solution

LunaP processes lunar optical imagery through a modular AI/ML perception pipeline, converts the resulting terrain information into a unified risk representation, and feeds that representation into mission-planning algorithms. 

The system supports:
*   Risk-aware route generation between user-defined waypoints.
*   Ranking of candidate landing or staging regions.
*   An interactive dashboard exposing the image, analysis, risk map, route, site ranking, and risk/confidence information.

## AI/ML Approach

The project deliberately avoids claiming a fully supervised deep-learning detector when the available public imagery is not guaranteed to have exhaustive hazard labels. Instead, the perception layer is modular and can combine:
1.  Available annotations
2.  Pretrained visual representations
3.  Automated terrain indicators
4.  Lightweight learning methods

This allows the prototype to demonstrate genuine ML-assisted terrain understanding without requiring the team to manually label thousands of images.

### Workflow
1.  **Training/Development:** Public lunar imagery → Preprocessing → Available annotations/metadata or automated terrain indicators → Feature learning/representation → Validation → Perception component.
2.  **Runtime:** New lunar image → Trained/pretrained perception component → Terrain features → Terrain-risk map → Risk-aware route planning + Landing-site ranking → Regions of Scientific Interests → Dashboard.

## Data Sources

Primary sources identified for the prototype include:
*   **Chandrayaan-2 Imagery:** Via the Indian Space Research Organisation/ISSDC ecosystem.
    *   [ISSDC Chandrayaan Data Explorer](https://chmapbrowse.issdc.gov.in/)
    *   [Kaggle — OHRC Images (Chandrayaan-2)](https://www.kaggle.com/datasets/piyushsharma5654/ohrc-images-chandaryaan-2)
*   **Lunar Reconnaissance Orbiter (LROC):** Accessible through LROC QuickMap.
    *   [LROC QuickMap](https://quickmap.lroc.im-ldi.com/)
*   **NASA CGI Moon Kit:** Provides LROC-derived color and LOLA elevation products for visualization.
    *   [NASA SVS — CGI Moon Kit](https://svs.gsfc.nasa.gov/4720)
*  **More Datasets are being explored to solve the problem statement more efficiently.**

## Core System Flow

`Dataset / public lunar imagery` → `Preprocessing` → `AI/ML perception development` → `Trained/pretrained perception component` → `New lunar image` → `Terrain perception` → `Hazard/risk representation` → `Mission planning` → `Safe route generation + Landing-site ranking` → `ROI Finding (Craters, Basins, Ridges, Boulders)` →  `Interactive mission dashboard`

## Technology Stack

*   **Language:** Python
*   **Image Processing:** OpenCV, NumPy
*   **Machine Learning:** PyTorch (or another lightweight ML framework)
*   **Optimization:** Numerical/graph-based optimization for route planning
*   **Interface:** Streamlit for the interactive dashboard

## Prototype Scope

*   Upload and process lunar surface imagery.
*   Generate a visual terrain/hazard representation.
*   Allow users to specify navigation waypoints.
*   Generate a route that accounts for terrain risk.
*   Identify and rank promising landing/staging regions.
*   Find suitable regions of interest.
*   Present the complete result through a lightweight interactive dashboard.

## Limitations and Honest Scope

The hackathon prototype is a **research demonstrator and decision-support system**, not flight-certified rover software. 
*   Optical imagery, lighting, shadows, image quality, and limited labelled data can affect perception accuracy.
*   Navigation quality depends on the terrain representation produced by the perception layer.
*   The proposal emphasizes a defensible end-to-end prototype rather than claiming flight-ready autonomy.

## Expected Outcome

A working proof of concept showing that lunar imagery can be transformed into terrain intelligence and then used to support autonomous mission planning. The key demonstration is: **understand the terrain, quantify risk, and choose a safer path.**
