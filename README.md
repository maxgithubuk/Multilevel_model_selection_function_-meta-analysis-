# Multilevel_model_selection_function_-meta-analysis-

#Multilevel meta-analytic models are preferable to two-level models when individual studies contribute multiple effect sizes, such as with repeated measurements over time, multiple instruments assessing the same outcome, or studies involving more than two groups. By explicitly modeling within-study dependencies, multilevel approaches estimate both within-study error and between-study variance, providing a more accurate representation of the data’s hierarchical structure.

#Using the metafor package’s rma.mv function alongside clubSandwich, multilevel models with cluster-robust standard errors can be easily computed. However, due to the reliance on Satterthwaite-adjusted degrees of freedom, these models become unstable when the degrees of freedom fall below four. In such cases, a two-level model is preferable, aggregating within-study dependencies between effects using an imputed rho value and constraining variances at the between-study level.

#This R function, fit_multilevel_or_twolevel_model, is designed for researchers and data analysts working on hierarchical meta-analyses, particularly those involving longitudinal studies or complex hierarchical datasets.It automates the selection and fitting of multilevel or two-level models, depending on the characteristics of the data. It also calculates heterogeneity and prediction intervals for pooled effects, summarizing them into lists.

Here’s a step-by-step explanation of the fit_multilevel_or_twolevel_model function, describing how it works and what each step accomplishes:

1. Define Timepoint Domains

The function begins by defining a list of timepoint categories (timepoint_domains). These categories specify groups of timepoints for inclusion or exclusion during data filtering:
	•	Example categories: “End of Treatment (EOT)”, “Follow-up (3, 6, 12 months)”, etc.
	•	Purpose: Flexibility to filter data based on timepoint criteria.

2. Validate Input Parameters
	•	The function checks if the provided timepoint_category is valid (matches one of the predefined categories in timepoint_domains).
	•	Error Handling: If invalid, it stops execution and provides an error message.

3. Filter Data
	•	Study Filtering: The function removes studies listed in the excluded_studies parameter.
	•	Timepoint Filtering: Based on the provided timepoint_category, it either includes or excludes specific timepoints:

4. Compute Covariance Matrix
	•	Covariance Matrix (V): Using impute_covariance_matrix, the function computes the covariance matrix for the effect sizes (es), accounting for clustering (study) and a specified correlation (rho).
	•	Purpose: This matrix captures within-study dependencies for accurate model fitting.

5. Fit Multilevel Model
	•	Optimizer Fallback Mechanism:
	•	The function first tries fitting a multilevel model using rma.mv with the nlminb optimizer.
	•	If nlminb fails, it switches to the optim optimizer.
	•	If both fail, the function stops execution and returns an error.
	•	Purpose: To ensure the multilevel model is robustly fitted to the data.

6. Apply Egger’s Test (Optional Diagnostic)
	•	Performs Egger’s regression test using the standard error (se) as a moderator to assess publication bias.
	•	Fallback Handling: If Egger’s test fails, it gracefully returns NULL.

7. Compute Heterogeneity
	•	The function calculates heterogeneity metrics (e.g., I²) using a custom utility, calculate_heterogeneity.

8. Robust Adjustment
	•	Uses clubSandwich methods to compute cluster-robust standard errors and adjust the degrees of freedom (Satterthwaite-adjusted).
	•	Check for Stability: If the adjusted degrees of freedom (df_Satt) fall below four, the multilevel model is deemed unstable, and the function transitions to a two-level approach.

9. Two-Level Model Fallback

If the multilevel model is unstable:
	1.	Pooling and Aggregation:
	•	The function aggregates effect sizes using pool_and_update and accounts for within-study dependencies via an imputed rho value.
	•	Handles repeated measures (e.g., over time) with specialized methods like CS*CAR (Compound Symmetry + Continuous AutoRegressive).
	2.	Two-Level Model Fitting:
	•	A simpler two-level model is fitted using rma with the aggregated data.
	•	Optimizer fallback (nlminb → optim) ensures robust fitting.
	3.	Egger’s Test for Two-Level Model: If applicable, Egger’s regression test is run on the two-level model.
	4.	Heterogeneity Metrics: Returns heterogeneity information specific to the two-level model (total I², etc.).

10. Output Results

Depending on whether a multilevel or two-level model is used:
	•	Multilevel Model Outputs:
	•	The robust model object (robust_mlm_model).
	•	Egger’s test results.
	•	Heterogeneity metrics.
	•	Predictions from the fitted model.
	•	Two-Level Model Outputs:
	•	Two-level model object.
	•	Egger’s test results (if applicable).
	•	Heterogeneity metrics.
	•	Predictions from the two-level model.

This structure ensures adaptability and robustness when analyzing hierarchical meta-analytic datasets.

Assumptions and Data Preprocessing Requirements
	1.	Data Structure:
	•	The input data must be a structured dataset (e.g., a data frame or tibble) that contains the following variables:
	•	Effect sizes (es): The primary dependent variable for meta-analysis.
	•	Variance (var): Variance of the effect sizes, required for computing the covariance matrix.
	•	Study identifier (study): A variable indicating unique study clusters.
	•	Effect size identifier (es.id): A variable to uniquely identify effect sizes within studies (for multilevel modeling).
	•	Timepoint string (timepoint_string): Categorical variable indicating the timepoint associated with each observation.
	2.	Variance Values:
	•	Variance (var) must be non-negative, as it is used to compute the covariance matrix.
	•	If variances are missing, they should be imputed or removed before running the function.
	3.	Timepoint Categories:
	•	The timepoint_string variable must correspond to one of the predefined categories in the timepoint_domains list.
	•	Invalid or unmatched categories should be excluded or recoded to match these domains.
	4.	Study and Effect Size Identifiers:
	•	The study variable must uniquely identify clusters (e.g., individual studies).
	•	The es.id variable is necessary to model the nested structure within studies.
	5.	Excluded Studies:
	•	The excluded_studies parameter is used to filter out specific studies. Ensure this corresponds to the identifiers in the study variable.
	6.	Sufficient Sample Size:
	•	The function assumes sufficient sample size within each study and timepoint to compute the required statistics reliably. Studies with too few observations might cause model fitting errors.
	7.	Covariance Matrix Assumptions:
	•	The function uses rho (correlation) and phi (autocorrelation) as parameters for covariance matrix computation. Ensure these are appropriately set for your data.
	9.	Effect Size Transformation:
	•	If the effect sizes (es) are not standardized, the user should ensure appropriate transformation (e.g., converting raw mean differences into standardized mean differences) before using the function, for instance by using the escalc function in metafor.
	11.	Repeated Measures Detection:
	•	The function expects that repeated measures (multiple timepoints for the same study) are coded appropriately in timepoint_string and detected automatically using group_by and summarize.


