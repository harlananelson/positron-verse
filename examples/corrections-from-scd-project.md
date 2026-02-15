# Positron-Verse Corrections — SCDCernerProject Code Review

> **Source:** `projects/SCDCernerProject/diseaseclassification/335-ML-Mortality-Improved.txt`
> (patterns also appear in 320, 325, 330, 340, 345, 400, 500, 510)

Each section shows the **original code** then the **corrected code** with the
rule violated. Use these as a teaching reference for applying positron-verse
conventions to real analysis code.

---

## 1. Package Loading: `pacman::p_load()` → explicit `library()`

**Rule:** `style-conventions.md` §1 — Load ALL packages at the top with `library()`.
`pacman::p_load` hides which packages are actually required and silently installs missing ones.

### Original
```r
pacman::p_load(
  tidyverse, data.table, magrittr, janitor, glue,
  tidymodels, ranger,
  yardstick, probably, pROC,
  vip,
  shapviz,
  survival, survminer,
  gt, gtsummary, knitr,
  targets, tarchetypes,
  IRdisplay
)

library(kernelshap)
```

### Corrected
```r
# Core data manipulation
library(tidyverse)
library(glue)

# ML framework
library(tidymodels)
library(ranger)

# Metrics and calibration
library(probably)
library(pROC)

# Variable importance and SHAP
library(vip)
library(kernelshap)
library(shapviz)

# Survival analysis
library(survival)
library(survminer)

# Tables and display
library(gt)
library(gtsummary)

# Pipeline
library(targets)

# File paths
library(here)
```

**Also removed:**
- `data.table` — not needed when using tidyverse (the code never uses data.table syntax)
- `magrittr` — loaded by tidyverse; use native pipe `|>` instead
- `janitor` — never called in this analysis
- `tarchetypes` — never called in this analysis
- `knitr` — only used for `kable()` which is available via `knitr::kable()` inline
- `IRdisplay` — Jupyter display utility, not needed in Quarto

---

## 2. Pipe: `%>%` → `|>`

**Rule:** `CLAUDE.md` Key Invariant 1 — Use `|>` not `%>%`.

### Original
```r
ads_raw <- tar_read(ads_with_features) %>%
  select(!where(~inherits(.x, "IDate")))

condition_features_100 <- ads_raw %>%
  summarize(across(all_of(conditionsFeatures_list), ~sum(.x, na.rm = TRUE))) %>%
  pivot_longer(everything(), names_to = "condition", values_to = "n_patients") %>%
  filter(n_patients >= 100) %>%
  pull(condition)
```

### Corrected
```r
ads_raw <- tar_read(ads_with_features) |>
  select(!where(\(x) inherits(x, "IDate")))

condition_features_100 <- ads_raw |>
  summarize(across(all_of(conditionsFeatures_list), \(x) sum(x, na.rm = TRUE))) |>
  pivot_longer(everything(), names_to = "condition", values_to = "n_patients") |>
  filter(n_patients >= 100) |>
  pull(condition)
```

**Note:** Every `~.x` lambda also changes to `\(x)` (see correction #3).

---

## 3. Lambda: `~.x` → `\(x)`

**Rule:** `style-conventions.md` §5 — Use modern lambda `\(x)` syntax, not purrr formula `~.x`.

### Original
```r
select(!where(~inherits(.x, "IDate")))
summarize(across(all_of(list), ~sum(.x, na.rm = TRUE)))
```

### Corrected
```r
select(!where(\(x) inherits(x, "IDate")))
summarize(across(all_of(list), \(x) sum(x, na.rm = TRUE)))
```

This applies throughout all 9 files — every `~` formula in `across()`, `map()`,
`where()`, etc.

---

## 4. File Paths: `file.path()` with env vars → `here::here()`

**Rule:** `CLAUDE.md` Key Invariant 5 — Use `here::here()` for paths, never
`setwd()`, never hand-built absolute paths.

### Original
```r
user_path <- Sys.getenv("SICKLE_CELL_DATA_PATH",
                        unset = file.path(path.expand("~"), "work", "Users", Sys.getenv("USER")))

project <- 'SickleCell'
dataLoc <- file.path(user_path, "inst", "extdata", project)
project_path <- file.path(user_path, "Projects", project)
output_dir <- file.path(project_path, "outputs")

source(file.path(project_path, "R", 'functions.R'))
source(file.path(user_path, 'R', 'lhn-functions.R'))
```

### Corrected
```r
# Project paths — here::here() anchors to project root (.Rproj or .here file)
output_dir <- here("outputs")
if (!dir.exists(output_dir)) dir.create(output_dir, recursive = TRUE)

# Source project functions
source(here("R", "functions.R"))
source(here("R", "lhn-functions.R"))
```

**Caveat:** This project uses a shared data path across multiple environments
(HPC, local). If `here()` can't reach the data, use a single config-driven path:

```r
# Alternative for shared data environments
data_root <- config::get("data_path")  # from config.yml
```

---

## 5. Assignment Style: `=` inside `mutate()` is fine, but string quotes should be consistent

**Rule:** `style-conventions.md` §3 — Use `<-` for assignment. Single quotes vs
double quotes: pick one and be consistent (tidyverse convention: double quotes).

### Original (mixed quotes)
```r
project <- 'SickleCell'
source(file.path(project_path, "R", 'functions.R'))
filter(gender %in% c('Female', 'Male'))
non_predictors <- c('personid', 'tenant', 'deceased', 'death_event')
```

### Corrected
```r
project <- "SickleCell"
source(here("R", "functions.R"))
filter(gender %in% c("Female", "Male"))
non_predictors <- c("personid", "tenant", "deceased", "death_event")
```

---

## 6. Variable Naming: camelCase → snake_case

**Rule:** `style-conventions.md` §2 — Use snake_case for all variables and functions.

### Original
```r
conditionsFeatures_list
biometricBaselineMedianFeatures_list
labs_factable
labs_factable_dedup
dataLoc
```

### Corrected
```r
conditions_features_list
biometric_baseline_median_features_list
labs_fact_table
labs_fact_table_dedup
data_loc
```

**Note:** These come from the targets pipeline, so renaming requires
coordinating with upstream `_targets.R`. When you can't rename the source,
assign to a snake_case alias:

```r
conditions_features <- conditionsFeatures_list
biometric_features <- biometricBaselineMedianFeatures_list
```

---

## 7. `cat()` for logging → `cli::cli_inform()` or structured output

**Rule:** `design-principles.md` §8 — Use cli for user-facing messages.

### Original
```r
cat("User path:", user_path, "\n")
cat("Total SCD patients after filtering:", nrow(data_full), "\n")
cat("Mortality rate:", round(mean(data_full$deceased == "died") * 100, 1), "%\n")
cat("\n=== DATA SPLIT SUMMARY ===\n")
cat("Main cohort (training/test):", nrow(data_main), "patients\n")
```

### Corrected
```r
cli::cli_h2("Data Split Summary")
cli::cli_inform("Main cohort (training/test): {.val {nrow(data_main)}} patients")
cli::cli_inform("External holdout: {.val {nrow(data_external)}} patients")
cli::cli_inform("Main mortality rate: {.val {round(mean(data_main$deceased == 'died') * 100, 1)}}%")
```

**In notebooks** (where output is rendered), `cat()` is acceptable for quick
diagnostics. But for functions that will be reused, prefer `cli`.

---

## 8. `ifelse()` → `if_else()` or `case_when()`

**Rule:** `anti-patterns.md` §4 — `ifelse()` is type-unstable. Use `dplyr::if_else()`.

### Original
```r
pred_class <- factor(
  ifelse(data[[prob_col]] >= optimal_threshold, "died", "censored"),
  levels = c("censored", "died")
)
```

### Corrected
```r
pred_class <- if_else(
  data[[prob_col]] >= optimal_threshold,
  "died",
  "censored"
) |>
  factor(levels = c("censored", "died"))
```

---

## 9. `as.formula(paste(...))` → formula directly or recipe pattern

**Rule:** `design-principles.md` §6 — Prefer tidy evaluation. Avoid string-based
formula construction.

### Original
```r
recipe(as.formula(paste(outcome, "~ .")), data = model_data) %>%
  update_role(personid, tenant, new_role = "id")
```

### Corrected
```r
recipe(model_data) |>
  update_role(all_of(predictors), new_role = "predictor") |>
  update_role(personid, tenant, new_role = "id") |>
  update_role(all_of(outcome), new_role = "outcome")
```

Or if the formula pattern is preferred, use `reformulate()`:

```r
recipe(reformulate(".", response = outcome), data = model_data) |>
  update_role(personid, tenant, new_role = "id")
```

---

## 10. Model Spec: `mode` inside `rand_forest()` → `set_mode()`

**Rule:** `method-decision-tree.md` §4 — Use the parsnip pattern:
Specify → Set Engine → Set Mode.

### Original
```r
rf_model <- rand_forest(
  mode = "classification",
  trees = 500,
  mtry = 11,
  min_n = 20
) %>%
  set_engine("ranger", importance = "impurity")
```

### Corrected
```r
rf_spec <- rand_forest(
  trees = 500,
  mtry = 11,
  min_n = 20
) |>
  set_engine("ranger", importance = "impurity") |>
  set_mode("classification")
```

**Also:** Name it `rf_spec` (specification), not `rf_model` (which implies it's
already fitted).

---

## 11. `kable()` → `gt()` for rendered tables

**Rule:** `method-decision-tree.md` §3 — Use `gt` for publication tables.

### Original
```r
cv_metrics_A %>%
  select(.metric, mean, std_err) %>%
  mutate(mean = round(mean, 4), std_err = round(std_err, 4)) %>%
  kable()
```

### Corrected
```r
cv_metrics_A |>
  select(.metric, mean, std_err) |>
  gt() |>
  fmt_number(columns = c(mean, std_err), decimals = 4) |>
  tab_header(title = "Cross-Validation Results (5-fold stratified)")
```

---

## 12. `group_by() |> summarize()` → `.by` for simple cases

**Rule:** `tidyverse-skill.md` §1 — Per-operation grouping with `.by` is
preferred for simple single-step summaries (dplyr 1.1+).

### Original
```r
data_external %>%
  group_by(tenant) %>%
  summarize(
    n_patients = n(),
    n_deaths = sum(deceased == "died"),
    mortality_pct = round(mean(deceased == "died") * 100, 1)
  )
```

### Corrected
```r
data_external |>
  summarize(
    n_patients = n(),
    n_deaths = sum(deceased == "died"),
    mortality_pct = round(mean(deceased == "died") * 100, 1),
    .by = tenant
  )
```

**When to keep `group_by()`:** When you need persistent groups across multiple
operations (e.g., `group_by() |> mutate() |> summarize()`).

---

## 13. ggplot2: Missing `labs()`, inconsistent structure

**Rule:** `ggplot2-skill.md` — One layer per line, always include `labs()`,
prefer `theme_minimal()`.

### Original
```r
ggplot(roc_data_A, aes(x = 1 - specificity, y = sensitivity, color = Dataset)) +
  geom_line(linewidth = 1) +
  geom_abline(slope = 1, intercept = 0, linetype = "dashed", color = "gray50") +
  labs(title = paste("Run A (WITH Age) - ROC Curves"),
       subtitle = paste0("Test AUC: ", round(metrics_test_A$auc, 3),
                         " | External AUC: ", round(metrics_external_A$auc, 3))) +
  theme_minimal()
```

### Corrected
```r
ggplot(roc_data_a, aes(x = 1 - specificity, y = sensitivity, color = dataset)) +
  geom_line(linewidth = 1) +
  geom_abline(slope = 1, intercept = 0, linetype = "dashed", color = "gray50") +
  scale_color_viridis_d(end = 0.8) +
  labs(
    title = "Run A (WITH Age) - ROC Curves",
    subtitle = glue(
      "Test AUC: {round(metrics_test_a$auc, 3)}",
      " | External AUC: {round(metrics_external_a$auc, 3)}"
    ),
    x = "1 - Specificity",
    y = "Sensitivity",
    color = "Dataset"
  ) +
  theme_minimal()
```

**Changes:** Explicit axis labels, `glue()` over `paste0()`, `scale_color_viridis_d()`
for colorblind safety, x/y labels in `labs()`.

---

## 14. `TRUE ~ ...` in `case_when()` → `.default`

**Rule:** `tidyverse-skill.md` §1 — Use `.default` in `case_when()` (dplyr 1.1+).

### Original
```r
case_when(
  str_detect(Variable, "loinc") ~ str_extract(Variable, "\\d+_\\d+$") |>
    str_replace("_", "-"),
  TRUE ~ NA_character_
)
```

### Corrected
```r
case_when(
  str_detect(variable, "loinc") ~ str_extract(variable, "\\d+_\\d+$") |>
    str_replace("_", "-"),
  .default = NA_character_
)
```

---

## 15. `gt_to_html()` → Just `gt()` (let Quarto render)

**Rule:** `output-conventions.md` — Let the rendering engine handle display.

### Original
```r
perf_summary_A %>%
  gt() %>%
  tab_header(title = "...", subtitle = "...") %>%
  gt_to_html()
```

### Corrected
```r
perf_summary_a |>
  gt() |>
  tab_header(title = "...", subtitle = "...")
```

In Quarto, `gt` objects render natively. `gt_to_html()` was a workaround for
Jupyter's IR kernel display, which Quarto's native rendering doesn't need.

---

## 16. Repeated Feature-Filtering Pattern → Extract a Helper

**Rule:** `design-principles.md` §1.2 — Compose simple functions. DRY.

### Original (repeated 5 times)
```r
condition_features_100 <- ads_raw |>
  summarize(across(all_of(conditions_features), \(x) sum(x, na.rm = TRUE))) |>
  pivot_longer(everything(), names_to = "condition", values_to = "n_patients") |>
  filter(n_patients >= 100) |>
  pull(condition)

procedure_features_100 <- ads_raw |>
  summarize(across(all_of(procedure_features), \(x) sum(x, na.rm = TRUE))) |>
  pivot_longer(everything(), names_to = "procedure", values_to = "n_patients") |>
  filter(n_patients >= 100) |>
  pull(procedure)

# ... same pattern for biometrics, loinc, lab_binary
```

### Corrected
```r
#' Filter features to those present in >= min_patients
filter_features_by_prevalence <- function(data, feature_list, min_patients = 100) {
  data |>
    summarize(across(all_of(feature_list), \(x) sum(x, na.rm = TRUE))) |>
    pivot_longer(everything(), names_to = "feature", values_to = "n_patients") |>
    filter(n_patients >= min_patients) |>
    pull(feature)
}

condition_features_100 <- filter_features_by_prevalence(ads_raw, conditions_features)
procedure_features_100 <- filter_features_by_prevalence(ads_raw, procedure_features)
lab_binary_features_100 <- filter_features_by_prevalence(ads_raw, lab_features)
```

For the continuous features (biometrics, LOINC) that need a different counting
method (non-NA rather than sum), create a variant:

```r
#' Filter continuous features by non-missing count
filter_continuous_by_prevalence <- function(data, feature_list, min_patients = 100) {
  data |>
    select(all_of(feature_list)) |>
    summarize(across(everything(), \(x) sum(!is.na(x)))) |>
    pivot_longer(everything(), names_to = "feature", values_to = "n_patients") |>
    filter(n_patients >= min_patients) |>
    pull(feature)
}

biometric_features_100 <- filter_continuous_by_prevalence(ads_raw, biometric_features)
loinc_features_100 <- filter_continuous_by_prevalence(ads_raw, loinc_features)
```

---

## 17. `strata = "deceased"` → `strata = deceased` (bare name)

**Rule:** tidymodels functions accept bare column names — string quoting is
unnecessary and inconsistent with the rest of the tidyverse.

### Original
```r
data_split <- initial_split(data_main, strata = "deceased", prop = 0.80)
cv_folds_A <- vfold_cv(train_A, v = 5, strata = deceased)  # inconsistent!
```

### Corrected
```r
data_split <- initial_split(data_main, strata = deceased, prop = 0.80)
cv_folds_a <- vfold_cv(train_a, v = 5, strata = deceased)
```

---

## 18. Object Naming: `_A`/`_B` Suffix → Descriptive Names

**Rule:** `style-conventions.md` §2 — Descriptive over brief.

### Original
```r
train_A, test_A, external_A, recipe_A, workflow_A, fitted_A, model_A
train_preds_A, test_preds_A, external_preds_A
metrics_test_A, metrics_external_A
cv_folds_A, cv_results_A, cv_metrics_A
perf_summary_A, site_metrics_A
```

### Corrected
```r
train_with_age, test_with_age, external_with_age
recipe_with_age, workflow_with_age, fit_with_age
test_preds_with_age, external_preds_with_age
metrics_test_with_age, metrics_external_with_age
cv_folds_with_age, cv_results_with_age
```

Or, for the dual-run pattern, use a list:

```r
runs <- list(
  with_age = list(predictors = predictors_with_age, label = "WITH Age"),
  without_age = list(predictors = predictors_without_age, label = "WITHOUT Age")
)

results <- map(runs, \(run) {
  rec <- create_recipe(train_data, run$predictors)
  wf <- workflow() |> add_recipe(rec) |> add_model(rf_spec)
  fitted <- fit(wf, data = train_data)
  list(workflow = wf, fit = fitted, label = run$label)
})
```

---

## Summary of Rules Applied

| # | Issue | Rule Source | Severity |
|---|-------|-----------|----------|
| 1 | `pacman::p_load()` → `library()` | style-conventions §1 | Medium |
| 2 | `%>%` → `\|>` | CLAUDE.md invariant | High |
| 3 | `~.x` → `\(x)` | style-conventions §5 | High |
| 4 | `file.path()` → `here()` | CLAUDE.md invariant | High |
| 5 | Mixed quote style | style-conventions §3 | Low |
| 6 | camelCase → snake_case | style-conventions §2 | Medium |
| 7 | `cat()` → `cli::cli_inform()` | design-principles §8 | Low |
| 8 | `ifelse()` → `if_else()` | anti-patterns §4 | Medium |
| 9 | String formula → `reformulate()` | design-principles §6 | Low |
| 10 | `mode` in spec → `set_mode()` | method-decision-tree §4 | Low |
| 11 | `kable()` → `gt()` | method-decision-tree §3 | Low |
| 12 | `group_by()` → `.by` | tidyverse-skill §1 | Low |
| 13 | Missing `labs()`, no palette | ggplot2-skill | Low |
| 14 | `TRUE ~` → `.default` | tidyverse-skill §1 | Low |
| 15 | `gt_to_html()` → just `gt()` | output-conventions | Low |
| 16 | Repeated pattern → helper | design-principles §1.2 | Medium |
| 17 | String strata → bare name | tidymodels conventions | Low |
| 18 | `_A`/`_B` → descriptive names | style-conventions §2 | Low |
