# Method Decision Tree — R Package Selection

**When to use which package/approach for common R tasks.**

---

## 1. Data Import

| Data Source | Package | Function | Notes |
|------------|---------|----------|-------|
| CSV/TSV | **readr** | `read_csv()`, `read_tsv()` | Always over `read.csv()` |
| Excel | **readxl** | `read_excel()` | No Java dependency (unlike xlsx) |
| SPSS/Stata/SAS | **haven** | `read_sav()`, `read_dta()`, `read_sas()` | Preserves labels |
| JSON | **jsonlite** | `fromJSON()` | |
| Google Sheets | **googlesheets4** | `read_sheet()` | Auth via gargle |
| Google Drive | **googledrive** | `drive_download()` | |
| Databases | **DBI** + backend | `dbConnect()`, `dbGetQuery()` | Use dbplyr for dplyr syntax |
| Arrow/Parquet | **arrow** | `read_parquet()`, `open_dataset()` | For large data |
| Web scraping | **rvest** | `read_html()` | |
| APIs | **httr2** | `request()`, `req_perform()` | httr2 over httr |

---

## 2. Data Manipulation

| Task | Package | Approach |
|------|---------|----------|
| Filter, select, mutate, summarize | **dplyr** | Core verbs with `|>` |
| Reshape (wide↔long) | **tidyr** | `pivot_longer()`, `pivot_wider()` |
| String manipulation | **stringr** | `str_*()` functions |
| Factor handling | **forcats** | `fct_*()` functions |
| Date/time | **lubridate** | `ymd()`, `dmy()`, parsing/arithmetic |
| Joins | **dplyr** | `left_join()`, `inner_join()`, etc. |
| Nested data | **tidyr** + **purrr** | `nest()`, `map()` |
| Row-wise operations | **dplyr** | `rowwise()` or `purrr::pmap()` |
| Database queries | **dbplyr** | Write dplyr, translates to SQL |
| Large data (>RAM) | **arrow** + **dplyr** | `open_dataset() |> filter() |> collect()` |

### Decision: dplyr vs base R

| Prefer dplyr | Prefer base R |
|--------------|---------------|
| Data frame transformations | Atomic vector operations |
| Grouped operations | Simple subsetting with `[` |
| Readable analysis pipelines | Performance-critical inner loops |
| Teaching/collaboration | Package internals (minimal dependencies) |

---

## 3. Visualization

| Task | Package | Notes |
|------|---------|-------|
| Static plots | **ggplot2** | Always the default |
| Interactive plots | **plotly** | `ggplotly()` for quick conversion |
| Animated plots | **gganimate** | Grammar of animated graphics |
| Network graphs | **ggraph** + **tidygraph** | Tidy network analysis |
| Composing plots | **patchwork** | `p1 + p2`, `p1 / p2` |
| Color palettes | **scales** | `scale_*_viridis_*()` built into ggplot2 |
| Maps | **sf** + **ggplot2** | `geom_sf()` |
| Tables | **gt** | Publication-quality tables |
| Summary tables | **gtsummary** | `tbl_summary()` for clinical/descriptive |

### ggplot2 Theme Selection

| Context | Theme |
|---------|-------|
| General purpose | `theme_minimal()` |
| Publication | `theme_classic()` |
| Presentation | `theme_minimal(base_size = 14)` |
| Custom branded | Build on `theme_minimal()` + `theme()` |

---

## 4. Modeling

| Task | Package | Approach |
|------|---------|----------|
| ML workflow orchestration | **tidymodels** | recipes → workflows → tune |
| Preprocessing/feature eng. | **recipes** | `step_*()` functions |
| Model specification | **parsnip** | Unified interface to model engines |
| Hyperparameter tuning | **tune** + **dials** | Grid search, Bayesian optimization |
| Resampling | **rsample** | `vfold_cv()`, `bootstraps()` |
| Model evaluation | **yardstick** | `roc_auc()`, `rmse()`, `brier_class()` |
| Variable importance | **vip** | `vip()`, `vi()` |
| Calibration | **probably** | `cal_plot_*()`, `cal_apply()` |
| Stacking/ensembles | **stacks** | Model stacking |
| Survival analysis | **censored** | Survival models in tidymodels |

### Model Engine Selection (via parsnip)

| Model Type | Engine | Package |
|-----------|--------|---------|
| Random forest | `ranger` | ranger |
| Gradient boosting | `xgboost` | xgboost |
| Logistic regression | `glm` | stats |
| Regularized regression | `glmnet` | glmnet |
| Neural network | `brulee` | brulee (torch) |
| SVM | `kernlab` | kernlab |
| Decision tree | `rpart` | rpart |

### tidymodels Canonical Workflow
```r
# 1. Split data
split <- initial_split(data, prop = 0.75, strata = outcome)
train <- training(split)
test  <- testing(split)

# 2. Recipe
rec <- recipe(outcome ~ ., data = train) |>
  update_role(id_cols, new_role = "id") |>
  step_dummy(all_nominal_predictors()) |>
  step_zv(all_predictors()) |>
  step_normalize(all_numeric_predictors())

# 3. Model specification
spec <- rand_forest(mtry = tune(), min_n = tune(), trees = 1000) |>
  set_engine("ranger") |>
  set_mode("classification")

# 4. Workflow
wf <- workflow() |>
  add_recipe(rec) |>
  add_model(spec)

# 5. Tune
folds <- vfold_cv(train, v = 10, strata = outcome)
tuned <- tune_grid(wf, resamples = folds, grid = 25)

# 6. Finalize and fit
best <- select_best(tuned, metric = "roc_auc")
final_wf <- finalize_workflow(wf, best)
final_fit <- last_fit(final_wf, split)

# 7. Evaluate
collect_metrics(final_fit)
```

---

## 5. Reporting and Publishing

| Task | Tool | Notes |
|------|------|-------|
| Documents/reports | **Quarto** (.qmd) | Successor to R Markdown |
| Notebooks | **Jupyter** (.ipynb) | Rendered by Quarto without re-execution |
| Presentations | **Quarto** (revealjs) | |
| Websites | **Quarto** (website project) | |
| Books | **Quarto** (book project) | |
| Package documentation | **pkgdown** | Static HTML docs |
| Function docs | **roxygen2** | Inline documentation |

---

## 6. Package Development

| Task | Package | Function |
|------|---------|----------|
| Create package | **usethis** | `create_package()` |
| Add function | Manual | Create .R file in `R/` |
| Document | **roxygen2** | `#'` comments → `devtools::document()` |
| Test | **testthat** | `usethis::use_test()` |
| Check | **devtools** | `devtools::check()` |
| Build website | **pkgdown** | `pkgdown::build_site()` |
| CI/CD | **usethis** | `use_github_action_check_standard()` |
| Dependencies | **usethis** | `use_package("dplyr")` |
| Version bumping | **usethis** | `use_version()` |

---

## 7. Project Infrastructure

| Task | Package/Tool | Notes |
|------|-------------|-------|
| File paths | **here** | `here::here("data", "raw.csv")` |
| Environment management | **renv** | `renv::init()`, `renv::snapshot()` |
| Configuration | **config** | `config::get()` |
| Pipeline orchestration | **targets** | Reproducible computation pipelines |
| Logging | **logger** | `log_info()`, `log_warn()` |
| CLI tools | **cli** | Beautiful console output |

---

## 8. Decision: When NOT to Use Tidyverse

| Situation | Recommendation |
|-----------|---------------|
| Package with zero dependencies | Use base R |
| Performance-critical loop body | Use base R or Rcpp |
| S4 class system (Bioconductor) | Follow Bioconductor conventions |
| Data.table user preference | Respect — it's a valid choice |
| Simple one-off vector ops | Base R is fine: `x[x > 0]` |
