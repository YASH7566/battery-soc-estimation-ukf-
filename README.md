# Unscented Kalman Filter for Li-ion Battery SOC Estimation

State-of-Charge (SOC) estimation for an LG 18650HG2 Li-ion cell using an Unscented Kalman Filter (UKF) built around a temperature- and SOC-dependent 2nd-order Equivalent Circuit Model (ECM), implemented in Simulink. The filter is validated against three standard EV drive cycles: UDDS, US06, and LA92.

## Overview

- **Cell:** LG 18650HG2, 3 Ah NMC chemistry
- **Model:** 2D (SOC × temperature) 2nd-order ECM — parameters identified from HPPC test data
- **Estimator:** Unscented Kalman Filter, implemented in Simulink (`UKF_SOC_Estimation.slx`)
- **Validation:** UDDS, US06, LA92 drive cycles

## Dataset

This project uses the **LG 18650HG2 Li-ion Battery Data**, collected at McMaster University, Hamilton, Ontario, by Dr. Phillip Kollmeyer. A brand-new 3 Ah LG HG2 cell was tested in an 8 cu.ft. thermal chamber across a range of ambient temperatures, including HPPC pulse tests and standard drive cycles (UDDS, US06, LA92, HWFET, and others).

- Dataset: [LG 18650HG2 Li-ion Battery Data — Mendeley Data](https://data.mendeley.com/datasets/cp3473x7xv/3) (DOI: 10.17632/cp3473x7xv.3)
- Citation:
  > P. Kollmeyer, C. Vidal, M. Naguib, M. Skells, "LG 18650HG2 Li-ion Battery Data and Example Deep Neural Network xEV SOC Estimator Script," Mendeley Data, 2020.

If you use this dataset, please cite it as instructed on the Mendeley page.

## Methodology

### 1. Parameter Extraction (HPPC)

Model parameters (R0, R1, C1, R2, C2, and OCV) were extracted from the HPPC test data using MATLAB's HPPC and ECM Parameterization objects/apps, resolved as functions of both SOC and temperature.

### 2. Battery Model

A **2nd-order Thevenin ECM** (two RC branches) was built as a 2D lookup-table model, with all resistive/capacitive parameters and OCV varying with SOC and temperature. This gives the model enough fidelity to track transient voltage response under aggressive drive-cycle loading while remaining computationally light enough for real-time-style filtering.

### 3. UKF Formulation

- **State vector:** SOC and RC-branch polarization voltages
- **Process model:** Coulomb counting for SOC propagation + RC branch dynamics
- **Measurement model:** Terminal voltage from the 2nd-order ECM (OCV − RC branch voltages − IR drop)
- **Sigma points:** Standard scaled unscented transform (α, β, κ tuned for this state dimension)

## Results

| Drive Cycle | RMSE | MAE | Max AE |
|---|---|---|---|
| UDDS  | 0.0114 | 0.0095 | 0.0495 |
| US06  | 0.0189 | 0.0161 | 0.0475 |
| LA92  | 0.0129 | 0.0109 | 0.0415 |

## Key Challenges & Fixes

- **Observability collapse:** Using voltage as the sole measurement channel left SOC weakly observable under low-excitation conditions. Fixed by adding a Coulomb-counting-derived second measurement channel, which stabilized convergence and materially improved RMSE.

Each of these was diagnosed at the model/file level (rather than by empirically tuning Q/R) before being fixed.

## Requirements

- MATLAB / Simulink (R2021b or later recommended)
- Simulink Control Design / System Identification-related toolboxes used for HPPC parameterization (if regenerating ECM parameters from raw data)

## How to Run

1. Clone the repo and download the LG 18650HG2 dataset from the link above into `data/`.
2. (Optional) Regenerate ECM parameters from the HPPC test data using MATLAB's HPPC/ECM Parameterization objects.
3. Open `UKF_SOC_Estimation.slx` in Simulink.
4. Load a drive cycle (UDDS/US06/LA92) as the input current/voltage profile.
5. Run the model; RMSE/MAE/MaxAE are computed and displayed against the logged reference SOC.

## Limitations & Future Work

- Validated only on the LG 18650HG2 cell and the temperature range covered by this dataset; generalization to other chemistries/cells is untested.

## References

1. UKF-based SOC estimation with a 2nd-order Thevenin ECM for battery pack equalization: [ETASR](https://etasr.com/index.php/ETASR/article/view/3111)
2. Modified UKF with improved parameter identification for 2nd-order ECM: [ScienceDirect](https://www.sciencedirect.com/science/article/pii/S1452398124001159)
3. Strong Tracking UKF with multiple suboptimal fading factors, 2nd-order RC ECM, validated on ECE15/UDDS: [Springer](https://link.springer.com/article/10.1007/s12239-024-00093-9)
