# Quantify-Uncertainty-of-Geospatial-Predictions
# Geospatial Uncertainty Quantification: A Unified Framework

This repository contains the codebase for my thesis project, which proposes a unified framework combining **spatially adaptive Conformal Prediction (CP)** with **Area-of-Applicability (AOA)** methods to rigorously quantify predictive uncertainty in geospatial machine learning models. 

By addressing the core challenges of spatial heterogeneity and covariate shift, this project provides a robust methodology for generating valid prediction intervals in unobserved geographical regions (extrapolation), ensuring that models accurately represent their own uncertainty in unknown spaces.

---

## Key Features & Novel Contributions

* **Geodetic Accuracy with Vincenty Distance**: Replaces traditional flat 2D Euclidean distance with Vincenty's formulae on the WGS-84 ellipsoid. This ensures near-millimeter accuracy for spatial decay weighting over vast and diverse geographical extents.
* **Topology-Driven Adaptive Bandwidth**: Implements a parameter-free, adaptive spatial bandwidth using Delaunay triangulations and Voronoi tessellations. By traversing a 2-hop topological neighborhood, the algorithm dynamically contracts the bandwidth in dense urban clusters and expands it in sparse rural areas, maintaining statistical efficiency without artificial boundary padding.
* **High-Fidelity Spatial Simulation**: Integrates directly with the Google Earth Engine (GEE) API to extract 64-dimensional satellite embeddings (`V1/ANNUAL`). It uses Principal Component Analysis (PCA) to extract a 1D structural prior, which then guides Fast Fourier Transform (FFT) Gaussian Random Fields to generate realistic, topographically consistent synthetic datasets for controlled stress-testing.

---

## Project Architecture & Pipeline

The codebase is structured into two core experimental phases, moving from real-world application to highly controlled synthetic stress-testing:

### 1. Real-World Application: Adaptive Conformal Prediction (King County)
The framework is first implemented and evaluated on the real-world King County (Seattle) dataset. This phase establishes the core mathematical and structural upgrades of the pipeline:
* **Spatial Embeddings Extraction**: Enriches the standard dataset by extracting 64-dimensional satellite embeddings via Google Earth Engine (GEE) to capture the underlying structural and environmental context of the region.
* **Enhanced CP Models**: Implements both the baseline **GeoCP** (relying solely on geographic proximity) and the novel **GeoSIMCP** (balancing geographic distance with feature-space similarity via MND).
* **Geodetic Accuracy & Kernel Testing**: Upgrades spatial distance calculations by replacing 2D Euclidean metrics with the highly accurate **Vincenty distance** on the WGS-84 ellipsoid. Furthermore, it systematically evaluates **4 different spatial decay kernels** (Gaussian, Exponential, Epanechnikov, Bisquare) to optimize calibration weights.
* **Topology-Driven Bandwidth**: Introduces an adaptive, parameter-free spatial bandwidth mechanism driven by **Voronoi regions** and Delaunay triangulations, allowing the model's spatial awareness to naturally expand or contract based on local point density.

### 2. Data Generation Process (DGP) & Covariate Shift Simulation
To rigorously validate the framework's reliability under extreme spatial heterogeneity, a custom Data Generation Process (DGP) was developed to synthesize phenomena and stress-test the models:
* **High-Fidelity Simulation**: Fetches GEE embeddings for highly polarized real-world regions (e.g., Paraisópolis vs. Morumbi in São Paulo). It applies Principal Component Analysis (PCA) to compress these embeddings into a 1D spatial skeleton, which then guides the generation of stationary Gaussian Random Fields (GRFs) via Fast Fourier Transform (FFT). These are blended and infused with aleatoric noise to create topographically consistent synthetic datasets.
* **Spatial Splitting (Covariate Shift)**: The generated datasets are subjected to distinct splitting paradigms to evaluate model performance under varying degrees of spatial heterogeneity:
  * **Random Split (Interpolation)**: Standard random sampling where the test set follows the same underlying distribution as the training set (exchangeability holds).
  * **Structured Split (Extrapolation)**: The map is strictly divided geographically (e.g., East/West). The model trains on one region and predicts on an entirely unseen region, forcing a severe spatial covariate shift to test the limits of the Conformal Prediction framework.
---

## Evaluation Metrics

The framework evaluates the generated prediction intervals using strict statistical metrics:
* **Empirical Coverage**: The percentage of true values falling within the predicted bounds.
* **Mean Interval Width**: The sharpness of the uncertainty bounds (narrower is better, provided coverage is met).
* **Mean Interval Score (Winkler Score)**: A composite metric that rewards narrow intervals but heavily penalizes intervals that fail to cover the true value.
* **Pearson Uncertainty Correlation**: Measures the model's self-awareness (correlation between interval width and actual absolute error).

---

## Setup & Dependencies

To run this code, ensure you have an active Google Earth Engine account and the following Python packages installed:

```bash
pip install numpy pandas matplotlib scikit-learn xgboost scipy geopy earthengine-api tqdm

import ee
ee.Authenticate()
ee.Initialize(project='your-project-id')
