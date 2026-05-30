Data are publicly available from Kaggle:

Non-Alcohol Fatty Liver Disease (NAFLD) Dataset

Source:
https://www.kaggle.com/datasets/utkarshx27/non-alcohol-fatty-liver-disease

Files used:
- nafld1.csv
- nafld2.csv

The raw datasets are not included in this repository.
Overview

This project examines longitudinal changes in HDL cholesterol levels among individuals with Non-Alcoholic Fatty Liver Disease (NAFLD) using repeated clinical measurements collected over time.

The analysis applies advanced longitudinal data analysis techniques, including:

Linear Mixed-Effects Models (LMM)
Generalized Estimating Equations (GEE)
Covariance Structure Comparison
Repeated Measures Analysis
Research Question

How do HDL cholesterol levels change over time among individuals with NAFLD, and are these changes associated with age, sex, and BMI?

Dataset

Publicly available NAFLD dataset obtained from Kaggle.

Files:

nafld1.csv (baseline participant characteristics)
nafld2.csv (longitudinal laboratory measurements)
Methods
Data Preparation
Random 10% sample selection
Data cleaning and merging
Long-format longitudinal dataset construction
HDL measurement extraction
Statistical Analysis
Descriptive Statistics
Linear Mixed-Effects Models
Covariance Structure Comparison
Compound Symmetry
AR(1)
Unstructured
Generalized Estimating Equations
Software
SAS
PROC MIXED
PROC GENMOD
PROC SURVEYSELECT
Key Findings
Female participants had significantly higher HDL levels than males.
Higher BMI was associated with lower HDL cholesterol levels.
Older age was associated with slightly higher HDL levels.
HDL levels showed a small increase over follow-up time.
AR(1) covariance structure provided the best model fit.
Results

Sample Size:

Participants: 1,125
HDL Observations: 13,875

Significant Predictors:

Follow-up Time
Age
Sex
BMI
