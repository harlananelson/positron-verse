# Tidymodels Skill

> **Usage:** Load this file into Claude Code (via CLAUDE.md pointer) or paste into an LLM session for building machine learning workflows following tidymodels conventions.

Comprehensive reference for the tidymodels ecosystem: rsample, recipes, parsnip, workflows, tune, dials, yardstick, broom, probably, vip, and stacks. Based on Max Kuhn and Julia Silge's *Tidy Modeling with R* (https://www.tmwr.org/).

---

## 1) The tidymodels Workflow

The canonical order:

1. **Split** data (rsample)
2. **Specify** preprocessing (recipes)
3. **Specify** model (parsnip)
4. **Bundle** into workflow (workflows)
5. **Tune** hyperparameters (tune + dials)
6. **Evaluate** performance (yardstick)
7. **Finalize** and fit (workflows)

---

## 2) Data Splitting (rsample)

```r
library(tidymodels)

# Train/test split
set.seed(123)
split <- initial_split(data, prop = 0.75, strata = outcome)
train <- training(split)
test  <- testing(split)

# Cross-validation
folds <- vfold_cv(train, v = 10, strata = outcome)

# Repeated CV
folds <- vfold_cv(train, v = 10, repeats = 5, strata = outcome)

# Bootstrap
boots <- bootstraps(train, times = 1000, strata = outcome)

# Time series
splits <- sliding_period(data, date_col, period = "month", lookback = 12)

# Validation set (alternative to CV)
val_split <- initial_validation_split(data, prop = c(0.6, 0.2), strata = outcome)
train <- training(val_split)
val   <- validation(val_split)
test  <- testing(val_split)
```

**Always stratify** on the outcome variable for classification tasks.

---

## 3) Preprocessing (recipes)

### Recipe Creation
```r
rec <- recipe(outcome ~ ., data = train) |>
  # Role assignment (non-predictor columns)
  update_role(patient_id, new_role = "id") |>

  # Imputation (before transformations)
  step_impute_median(all_numeric_predictors()) |>
  step_impute_mode(all_nominal_predictors()) |>

  # Encoding
  step_dummy(all_nominal_predictors()) |>

  # Numeric transformations
  step_normalize(all_numeric_predictors()) |>  # center + scale
  step_log(skewed_var, base = 10) |>

  # Feature engineering
  step_interact(terms = ~ var1:var2) |>
  step_poly(continuous_var, degree = 2) |>
  step_ns(date_var, deg_free = 4) |>         # Natural splines

  # Feature selection
  step_zv(all_predictors()) |>                # Remove zero-variance
  step_corr(all_numeric_predictors(), threshold = 0.9) |>  # Remove correlated
  step_nzv(all_predictors())                  # Near-zero variance
```

### Step Ordering Best Practice
1. Imputation
2. Individual transformations (log, sqrt)
3. Discretization
4. Dummy variables / encoding
5. Interactions
6. Normalization (center/scale)
7. Feature reduction (PCA, etc.)
8. Zero-variance / near-zero-variance removal

### Common Step Functions

| Step | Purpose |
|------|---------|
| `step_impute_median()` | Numeric imputation |
| `step_impute_knn()` | KNN-based imputation |
| `step_impute_bag()` | Bagged tree imputation |
| `step_dummy()` | One-hot encode categoricals |
| `step_normalize()` | Center and scale |
| `step_log()` | Log transform |
| `step_YeoJohnson()` | Power transform |
| `step_pca()` | Principal components |
| `step_date()` | Extract date features |
| `step_holiday()` | Holiday indicators |
| `step_novel()` | Handle new factor levels |
| `step_other()` | Pool infrequent factor levels |
| `step_downsample()` | Class imbalance (themis) |
| `step_smote()` | Synthetic oversampling (themis) |

### Checking Recipe Results
```r
# Prep and bake for inspection
prepped <- prep(rec, training = train)
baked <- bake(prepped, new_data = train)
glimpse(baked)

# See what steps do
tidy(prepped)
tidy(prepped, number = 3)  # Details of step 3
```

---

## 4) Model Specification (parsnip)

### Pattern: Specify → Set Engine → Set Mode
```r
# Random forest
rf_spec <- rand_forest(
  mtry = tune(),
  min_n = tune(),
  trees = 1000
) |>
  set_engine("ranger", importance = "impurity") |>
  set_mode("classification")

# Logistic regression
lr_spec <- logistic_reg(
  penalty = tune(),
  mixture = tune()
) |>
  set_engine("glmnet") |>
  set_mode("classification")

# XGBoost
xgb_spec <- boost_tree(
  trees = tune(),
  tree_depth = tune(),
  min_n = tune(),
  learn_rate = tune(),
  loss_reduction = tune(),
  sample_size = tune()
) |>
  set_engine("xgboost") |>
  set_mode("classification")

# Linear regression
lm_spec <- linear_reg() |>
  set_engine("lm")

# Regularized regression
en_spec <- linear_reg(penalty = tune(), mixture = tune()) |>
  set_engine("glmnet")
```

### Available Models (selection)

| parsnip Function | Engines | Modes |
|-----------------|---------|-------|
| `linear_reg()` | lm, glmnet, stan | regression |
| `logistic_reg()` | glm, glmnet, stan | classification |
| `rand_forest()` | ranger, randomForest | both |
| `boost_tree()` | xgboost, lightgbm, C5.0 | both |
| `decision_tree()` | rpart, C5.0 | both |
| `svm_rbf()` | kernlab | both |
| `svm_poly()` | kernlab | both |
| `nearest_neighbor()` | kknn | both |
| `mlp()` | nnet, brulee | both |
| `proportional_hazards()` | survival, glmnet | censored regression |

Full list: https://www.tidymodels.org/find/parsnip/

---

## 5) Workflows

```r
# Bundle recipe + model
wf <- workflow() |>
  add_recipe(rec) |>
  add_model(rf_spec)

# Or with formula (no recipe needed for simple cases)
wf <- workflow() |>
  add_formula(outcome ~ predictor1 + predictor2) |>
  add_model(lm_spec)

# Or with variable selection
wf <- workflow() |>
  add_variables(outcomes = outcome, predictors = everything()) |>
  add_model(rf_spec)
```

---

## 6) Hyperparameter Tuning (tune + dials)

### Grid Search
```r
# Random grid (usually preferred)
tuned <- tune_grid(
  wf,
  resamples = folds,
  grid = 25,                    # 25 random parameter combinations
  metrics = metric_set(roc_auc, accuracy, brier_class),
  control = control_grid(save_pred = TRUE, verbose = TRUE)
)

# Regular grid
grid <- grid_regular(
  mtry(range = c(2, 10)),
  min_n(range = c(2, 40)),
  levels = 5
)
tuned <- tune_grid(wf, resamples = folds, grid = grid)
```

### Bayesian Optimization
```r
tuned <- tune_bayes(
  wf,
  resamples = folds,
  initial = 10,
  iter = 50,
  metrics = metric_set(roc_auc),
  control = control_bayes(no_improve = 15, verbose = TRUE)
)
```

### Racing Methods (faster)
```r
library(finetune)
tuned <- tune_race_anova(
  wf,
  resamples = folds,
  grid = 50,
  metrics = metric_set(roc_auc),
  control = control_race(verbose = TRUE)
)
```

### Inspecting Results
```r
show_best(tuned, metric = "roc_auc")
autoplot(tuned)
collect_metrics(tuned)
```

---

## 7) Finalizing and Fitting

```r
# Select best parameters
best_params <- select_best(tuned, metric = "roc_auc")

# Finalize workflow
final_wf <- finalize_workflow(wf, best_params)

# last_fit: fit on train, evaluate on test (one step)
final_fit <- last_fit(final_wf, split)

# Collect final metrics
collect_metrics(final_fit)

# Get test predictions
collect_predictions(final_fit)

# Extract fitted model for deployment
final_model <- extract_workflow(final_fit)

# Or fit on full training data for deployment
deployed_model <- fit(final_wf, data = train)
```

---

## 8) Model Evaluation (yardstick)

### Classification Metrics
```r
predictions |>
  roc_auc(truth = outcome, .pred_positive_class)

predictions |>
  conf_mat(truth = outcome, estimate = .pred_class) |>
  autoplot(type = "heatmap")

# Multiple metrics at once
metrics <- metric_set(roc_auc, accuracy, sensitivity, specificity, brier_class)
predictions |> metrics(truth = outcome, .pred_positive_class, estimate = .pred_class)

# ROC curve
predictions |>
  roc_curve(truth = outcome, .pred_positive_class) |>
  autoplot()
```

### Regression Metrics
```r
metrics <- metric_set(rmse, rsq, mae)
predictions |> metrics(truth = actual, estimate = .pred)
```

---

## 9) Variable Importance (vip)

```r
library(vip)

# From fitted workflow
final_fit |>
  extract_fit_parsnip() |>
  vip(num_features = 20)

# Model-agnostic (SHAP-like)
final_fit |>
  extract_fit_parsnip() |>
  vi()
```

---

## 10) Calibration (probably)

```r
library(probably)

# Calibration plot
predictions |>
  cal_plot_breaks(truth = outcome, .pred_positive_class)

# Calibration curve
predictions |>
  cal_plot_windowed(truth = outcome, .pred_positive_class)

# Apply calibration
cal_obj <- cal_estimate_logistic(predictions, truth = outcome)
calibrated <- cal_apply(new_predictions, cal_obj)
```

---

## 11) Comparing Multiple Models (workflowsets)

```r
library(workflowsets)

# Define multiple model specs
models <- list(
  rf  = rf_spec,
  xgb = xgb_spec,
  lr  = lr_spec
)

# Create workflow set
wf_set <- workflow_set(
  preproc = list(base = rec),
  models = models,
  cross = TRUE
)

# Tune all
results <- wf_set |>
  workflow_map(
    "tune_grid",
    resamples = folds,
    grid = 25,
    metrics = metric_set(roc_auc),
    verbose = TRUE
  )

# Compare
autoplot(results)
rank_results(results, rank_metric = "roc_auc")
```

---

## 12) Model Stacking (stacks)

```r
library(stacks)

stack <- stacks() |>
  add_candidates(rf_results) |>
  add_candidates(xgb_results) |>
  add_candidates(lr_results) |>
  blend_predictions(metric = metric_set(roc_auc)) |>
  fit_members()

predict(stack, new_data = test)
```

---

## 13) Key References

- **Tidy Modeling with R**: https://www.tmwr.org/
- **Feature Engineering and Selection**: http://www.feat.engineering/
- **tidymodels.org**: https://www.tidymodels.org/
- **Find models**: https://www.tidymodels.org/find/parsnip/
- **Find recipe steps**: https://www.tidymodels.org/find/recipes/
