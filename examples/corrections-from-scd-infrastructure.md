# Positron-Verse Corrections — SCDCernerProject Infrastructure Files

> **Sources:**
> - `projects/SCDCernerProject/R/_targets.R` (888 lines — targets pipeline)
> - `projects/SCDCernerProject/R/functions.R` (1924 lines — project functions)
> - `projects/SCDCernerProject/R/functions-lhn.R` (1061 lines — multi-project utilities)
> - `projects/SCDCernerProject/R/NEJM-Prod-Figures.R` (1095 lines — manuscript figures)

Each section shows the **original code** then the **corrected code** with the
rule violated. These complement `corrections-from-scd-project.md` (analysis
notebooks) with patterns found in the project's infrastructure layer.

---

## 1. Target Names: camelCase → snake_case

**Rule:** `style-conventions.md` §2 — Use `snake_case` for all identifiers.
Target names are referenced throughout the pipeline; inconsistent casing
creates confusion about which names are targets vs. local variables.

### Original (`_targets.R`)
```r
tar_target(encountersCSV, file.path(dataLoc, 'encounterExtract_SCD_RWD.csv'), format = 'file'),
tar_target(ADSpatientCSV, file.path(dataLoc, 'adspatient_SCD_RWD_2024_07_30.csv'), format = 'file'),
tar_target(comorbidity_ConditionEncounterCSV, file.path(dataLoc, 'comorbidity_ConditionEncounter_SCD_RWD.csv'), format = 'file'),
tar_target(Lab_LOINCEncounterFeatureCSV, file.path(dataLoc, 'Lab_LOINCEncounterFeature_SCD_RWD.csv'), format = 'file'),
tar_target(ADSpatient_raw, get_data(ADSpatientCSV)),
tar_target(ADSpatient, create_ADSpatient(ADSpatient_raw)),
tar_target(hydroxiaUsageTrue, create_hydroxiaUsageTrue(hydroxiaUsage)),
```

### Corrected
```r
tar_target(encounters_csv, file.path(data_loc, "encounterExtract_SCD_RWD.csv"), format = "file"),
tar_target(ads_patient_csv, file.path(data_loc, "adspatient_SCD_RWD_2024_07_30.csv"), format = "file"),
tar_target(comorbidity_condition_encounter_csv, file.path(data_loc, "comorbidity_ConditionEncounter_SCD_RWD.csv"), format = "file"),
tar_target(lab_loinc_encounter_feature_csv, file.path(data_loc, "Lab_LOINCEncounterFeature_SCD_RWD.csv"), format = "file"),
tar_target(ads_patient_raw, get_data(ads_patient_csv)),
tar_target(ads_patient, create_ads_patient(ads_patient_raw)),
tar_target(hydroxia_usage_true, create_hydroxia_usage_true(hydroxia_usage)),
```

---

## 2. Pipeline Variable Names: camelCase → snake_case

**Rule:** `style-conventions.md` §2 — Use `snake_case` for variables.

### Original (`_targets.R`)
```r
project <- 'SickleCell'
projectSchema <- 'sicklecell_study_rwd'
dataLoc <- file.path(user_path, "inst", "extdata", project)
project_path <- file.path(user_path, "Projects", project)
```

### Corrected
```r
project <- "SickleCell"
project_schema <- "sicklecell_study_rwd"
data_loc <- file.path(user_path, "inst", "extdata", project)
project_path <- file.path(user_path, "Projects", project)
```

---

## 3. Function Names: camelCase → snake_case

**Rule:** `style-conventions.md` §2 — Functions use `snake_case`.
Functions with camelCase parameters propagate inconsistency into every call
site and every target name that references them.

### Original (`functions.R`)
```r
create_ADSpatient <- function(data) { ... }
create_hydroxpatients <- function(hydroxpatients_raw) { ... }
create_hydroxiaUsageTrue <- function(hydroxiaUsage) { ... }
create_hydroxia <- function(hydroxiaUsageTrue, hydroxpatients) { ... }
```

### Corrected
```r
create_ads_patient <- function(data) { ... }
create_hydrox_patients <- function(hydrox_patients_raw) { ... }
create_hydroxia_usage_true <- function(hydroxia_usage) { ... }
create_hydroxia <- function(hydroxia_usage_true, hydrox_patients) { ... }
```

---

## 4. `T`/`F` → `TRUE`/`FALSE`

**Rule:** `anti-patterns.md` §2.1 — Never use `T`/`F`; they can be
overwritten by user assignment (`T <- 0`). Always spell out `TRUE`/`FALSE`.

### Original (`functions.R:537`)
```r
dplyr::mutate(hydroxyurea = if_else(!is.na(hydroxyurea), T, F)) %>%
```

### Corrected
```r
dplyr::mutate(hydroxyurea = if_else(!is.na(hydroxyurea), TRUE, FALSE)) |>
```

### Original (`functions-lhn.R:735`)
```r
    T ~ 'Not Hispanic'
  )
```

### Corrected
```r
    .default = "Not Hispanic"
  )
```

Note: This also fixes the `TRUE ~` anti-pattern (see correction #5).

---

## 5. `TRUE ~` in `case_when()` → `.default`

**Rule:** `anti-patterns.md` §3.4 — Use `.default` argument instead of
`TRUE ~ value` in `case_when()`. The `.default` parameter was added in
dplyr 1.1.0 and is the idiomatic approach.

### Original (`functions.R:1259`)
```r
transmute(
  end_organ_damage = case_when(
    `End Organ Damage` == "None" ~ "none",
    `End Organ Damage` == "Mild/Moderate" ~ "mild_moderate",
    `End Organ Damage` == "Severe Bone/Retina" ~ "severe_bone_retina",
    `End Organ Damage` == "Severe HLKB" ~ "severe_hlkb",
    TRUE ~ tolower(`End Organ Damage`)
  )
)
```

### Corrected
```r
transmute(
  end_organ_damage = case_when(
    `End Organ Damage` == "None" ~ "none",
    `End Organ Damage` == "Mild/Moderate" ~ "mild_moderate",
    `End Organ Damage` == "Severe Bone/Retina" ~ "severe_bone_retina",
    `End Organ Damage` == "Severe HLKB" ~ "severe_hlkb",
    .default = tolower(`End Organ Damage`)
  )
)
```

---

## 6. `stop()` → `cli::cli_abort()`

**Rule:** `design-principles.md` §8 — Use `cli::cli_abort()` for errors,
`cli::cli_warn()` for warnings, `cli::cli_inform()` for messages. These
provide structured error messages with cross-references and consistent
formatting.

### Original (`functions.R:238-242`)
```r
missing_cols <- setdiff(required_cols, names(labs))
if (length(missing_cols) > 0) {
  stop(
    "Missing required columns in labs data: ",
    paste(missing_cols, collapse = ", ")
  )
}
```

### Corrected
```r
missing_cols <- setdiff(required_cols, names(labs))
if (length(missing_cols) > 0) {
  cli::cli_abort(
    "Missing required columns in {.arg labs}: {.val {missing_cols}}."
  )
}
```

### Original (`functions-lhn.R:152-158`)
```r
if (!is.character(id_field) || length(id_field) != 1) {
  stop("id_field must be a single character string")
}

if (!(value_type %in% c("numeric", "categorical"))) {
  stop("value_type must be either 'numeric' or 'categorical'")
}
```

### Corrected
```r
if (!is.character(id_field) || length(id_field) != 1) {
  cli::cli_abort("{.arg id_field} must be a single character string, not {.obj_type_friendly {id_field}}.")
}

if (!(value_type %in% c("numeric", "categorical"))) {
  cli::cli_abort(
    "{.arg value_type} must be {.or {.val {c('numeric', 'categorical')}}}, not {.val {value_type}}."
  )
}
```

---

## 7. `stopifnot()` → `cli::cli_abort()` with informative message

**Rule:** `design-principles.md` §8 — `stopifnot()` produces cryptic error
messages. Use `cli::cli_abort()` with context about what went wrong.

### Original (`functions.R:1244`)
```r
load_shah_severity_matrix <- function(matrix_path) {
  stopifnot(file.exists(matrix_path))
  ...
}
```

### Corrected
```r
load_shah_severity_matrix <- function(matrix_path) {
  if (!file.exists(matrix_path)) {
    cli::cli_abort("Shah severity matrix not found at {.path {matrix_path}}.")
  }
  ...
}
```

---

## 8. `library()` mid-file → move to top

**Rule:** `style-conventions.md` §1 — Load ALL packages at the top of the
file. Mid-file `library()` calls are easy to miss and can cause confusing
load-order bugs.

### Original (`functions.R:1226-1228`)
```r
# ============================================================================
# SECTION 1: CONFIGURATION AND TRUTH TABLE
# ============================================================================

library(tidyverse)
library(lubridate)
library(data.table)
```

### Corrected
Remove these lines entirely — `functions.R` is sourced by `_targets.R`
which already loads these packages. If the file must work standalone, move
`library()` calls to the very top of the file (before the first function
definition on line 9).

---

## 9. `=` for function assignment → `<-`

**Rule:** `style-conventions.md` §3.1 — Always use `<-` for assignment,
never `=`.

### Original (`functions-lhn.R:769`)
```r
top_groups = function(.d, .group, levels = 3) {
  ...
}
```

### Corrected
```r
top_groups <- function(.d, .group, levels = 3) {
  ...
}
```

---

## 10. Explicit `return()` → implicit return

**Rule:** `style-conventions.md` §4 — Rely on R's implicit return.
`return()` is only needed for early exits. The last expression in a function
is automatically its return value.

### Original (`functions-lhn.R:17-33`)
```r
get_data <- function(file, cols = NULL) {
  if (is.null(cols)) {
    data <- data.table::fread(file)
  } else {
    data <- data.table::fread(file, select = cols)
  }
  cleaned_names <- janitor::make_clean_names(names(data))
  data.table::setnames(data, cleaned_names)
  return(data)
}
```

### Corrected
```r
get_data <- function(file, cols = NULL) {
  data <- if (is.null(cols)) {
    data.table::fread(file)
  } else {
    data.table::fread(file, select = cols)
  }
  cleaned_names <- janitor::make_clean_names(names(data))
  data.table::setnames(data, cleaned_names)
  data
}
```

---

## 11. `cat()` for logging → `cli::cli_inform()` / `cli::cli_alert()`

**Rule:** `anti-patterns.md` §1.5 — `cat()` produces unstructured output
that can't be suppressed. Use `cli::cli_inform()` for user-facing messages
or `cli::cli_alert_success()` for progress indicators.

### Original (`NEJM-Prod-Figures.R:68-69`)
```r
cat("=== Population Validation ===\n")
cat("Total N:", nrow(pop2), "\n")
```

### Corrected
```r
cli::cli_h2("Population Validation")
cli::cli_alert_info("Total N: {.val {nrow(pop2)}}")
```

### Original (`NEJM-Prod-Figures.R:770`)
```r
cat("✓ Figure 5 saved\n")
```

### Corrected
```r
cli::cli_alert_success("Figure 5 saved")
```

---

## 12. `sprintf()` → `glue::glue()`

**Rule:** `style-conventions.md` §5 — Prefer `glue::glue()` for string
interpolation. `sprintf()` with positional `%` placeholders is harder to
read and error-prone.

### Original (`NEJM-Prod-Figures.R:430-434`)
```r
forest_data <- results_fig3 %>%
  mutate(
    label = sprintf(
      "HR = %.2f (%.2f, %.2f)\nn = %d, events = %d\np = %s",
      hr, lower_ci, upper_ci, n, events,
      ifelse(p_value < 0.001, "<0.001", sprintf("%.3f", p_value))
    )
  )
```

### Corrected
```r
forest_data <- results_fig3 |>
  mutate(
    p_label = if_else(p_value < 0.001, "<0.001", format(round(p_value, 3), nsmall = 3)),
    label = glue::glue(
      "HR = {round(hr, 2)} ({round(lower_ci, 2)}, {round(upper_ci, 2)})\n",
      "n = {n}, events = {events}\n",
      "p = {p_label}"
    )
  )
```

---

## 13. `ifelse()` → `if_else()`

**Rule:** `anti-patterns.md` §4.2 — `ifelse()` strips attributes and
doesn't enforce type consistency. Use `dplyr::if_else()` which is
type-stable and preserves classes (dates, factors, etc.).

### Original (`NEJM-Prod-Figures.R:433`)
```r
ifelse(p_value < 0.001, "<0.001", sprintf("%.3f", p_value))
```

### Corrected
```r
if_else(p_value < 0.001, "<0.001", format(round(p_value, 3), nsmall = 3))
```

---

## 14. `map_df()` (deprecated) → `map() |> list_rbind()`

**Rule:** `anti-patterns.md` §3.2 — `map_df()` was deprecated in purrr 1.0.0.
Use `map() |> list_rbind()` for row-binding mapped results.

### Original (`NEJM-Prod-Figures.R:420`)
```r
results_fig3 <- map_df(thresholds, function(threshold) {
  run_threshold_analysis(pop2, threshold)
})
```

### Corrected
```r
results_fig3 <- map(thresholds, \(threshold) {
  run_threshold_analysis(pop2, threshold)
}) |>
  list_rbind()
```

---

## 15. `size` (deprecated) → `linewidth` in ggplot2

**Rule:** `ggplot2-skill.md` §1 — `size` for line width was deprecated
in ggplot2 3.4.0. Use `linewidth` for line-based geoms (`geom_line`,
`geom_step`, `geom_segment`, `geom_errorbar`).

### Original (`NEJM-Prod-Figures.R:700-705`)
```r
geom_segment(
  data = medians_fig4,
  aes(x = median, xend = median, y = 0, yend = 0.5),
  linetype = "dashed", size = 0.5, inherit.aes = FALSE
) +
geom_segment(
  data = medians_fig5,
  aes(x = median, xend = median, y = 0, yend = 0.5),
  linetype = "dashed", size = 0.5, inherit.aes = FALSE
)
```

### Corrected
```r
geom_segment(
  data = medians_fig4,
  aes(x = median, xend = median, y = 0, yend = 0.5),
  linetype = "dashed", linewidth = 0.5, inherit.aes = FALSE
) +
geom_segment(
  data = medians_fig5,
  aes(x = median, xend = median, y = 0, yend = 0.5),
  linetype = "dashed", linewidth = 0.5, inherit.aes = FALSE
)
```

### Original (`NEJM-Prod-Figures.R:308`)
```r
# survminer ggsurvplot call
size = 1.2,
```

### Corrected
```r
# survminer ggsurvplot call
size = 1.2,  # Note: survminer uses 'size' (maps to linewidth internally)
# For native ggplot2 geoms, use linewidth = 1.2 instead
```

---

## 16. Mixed quoting → consistent double quotes

**Rule:** `style-conventions.md` §3 — Use double quotes `"` consistently.
Single quotes `'` are acceptable in R but mixing is inconsistent.

### Original (`_targets.R` mixed throughout)
```r
project <- 'SickleCell'
projectSchema <- 'sicklecell_study_rwd'
targets::tar_option_set(packages = c('data.table', 'tidyverse'))
tar_target(encountersCSV, file.path(dataLoc, 'encounterExtract_SCD_RWD.csv'), format = 'file'),
```

### Corrected
```r
project <- "SickleCell"
project_schema <- "sicklecell_study_rwd"
targets::tar_option_set(packages = c("data.table", "tidyverse"))
tar_target(encounters_csv, file.path(data_loc, "encounterExtract_SCD_RWD.csv"), format = "file"),
```

---

## 17. `data.frame()` → `tibble()`

**Rule:** `method-decision-tree.md` §2 — Use `tibble::tibble()` for manual
data construction. Tibbles don't convert strings to factors, print cleanly,
and don't use partial matching.

### Original (common pattern across files)
```r
data.frame(
  group = c("SCD", "Control"),
  median = c(med_scd, med_control)
)
```

### Corrected
```r
tibble(
  group = c("SCD", "Control"),
  median = c(med_scd, med_control)
)
```

---

## 18. `gsub()` / `paste0()` → `str_replace()` / `glue()`

**Rule:** `method-decision-tree.md` §2 — Prefer stringr functions for
string manipulation and glue for interpolation. They are more readable
and consistent with the tidyverse.

### Original (`functions-lhn.R:91`)
```r
mutate(
  feature_name = paste0(prefix, '_', gsub('-', '_', .data[[codefield]]))
)
```

### Corrected
```r
mutate(
  feature_name = glue::glue("{prefix}_{str_replace_all(.data[[codefield]], '-', '_')}")
)
```

---

## 19. `cat()` for debug logging → remove or use `cli`

**Rule:** `anti-patterns.md` §1.5 — Debug logging with `cat()` should be
removed before production code, or replaced with `cli::cli_inform()` gated
on a `verbose` parameter.

### Original (`functions-lhn.R:244`)
```r
if (debug) {
  cat("Creating aggregated data with ranking...\n")
}
```

### Corrected
```r
if (debug) {
  cli::cli_inform("Creating aggregated data with ranking...")
}
```

---

## 20. `!!rlang::sym()` → `.data[[]]` or `pick()`

**Rule:** `tidyverse-skill.md` §1 — For dynamic column access by string,
prefer `.data[[col]]` (tidy evaluation) or `pick()` over the
bang-bang-sym pattern which is harder to read.

### Original (`functions-lhn.R:798-801`)
```r
if (sort_desc) {
  data <- data %>% dplyr::arrange(dplyr::desc(!!rlang::sym(sort_col)))
} else {
  data <- data %>% dplyr::arrange(!!rlang::sym(sort_col))
}
```

### Corrected
```r
if (sort_desc) {
  data <- data |> dplyr::arrange(dplyr::desc(.data[[sort_col]]))
} else {
  data <- data |> dplyr::arrange(.data[[sort_col]])
}
```

---

## 21. camelCase column names created in `mutate()` → snake_case

**Rule:** `style-conventions.md` §2 — Column names follow the same
`snake_case` convention as variable names.

### Original (`functions.R:197`)
```r
create_hydroxiaUsageTrue <- function(hydroxiaUsage) {
  hydroxiaUsage %>%
    janitor::clean_names() %>%
    dplyr::mutate(hydroxyureaUsage = TRUE)
}
```

### Corrected
```r
create_hydroxia_usage_true <- function(hydroxia_usage) {
  hydroxia_usage |>
    janitor::clean_names() |>
    dplyr::mutate(hydroxyurea_usage = TRUE)
}
```

### Original (`functions.R:579`)
```r
timetodeathafterlastEnounter = as.numeric(
```

### Corrected
```r
time_to_death_after_last_encounter = as.numeric(
```

---

## 22. `pacman::p_load()` in production scripts → explicit `library()`

**Rule:** `style-conventions.md` §1 — Already covered in
`corrections-from-scd-project.md` #1, but appears again in `NEJM-Prod-Figures.R`.
Critical for production/publication scripts where reproducibility is paramount.

### Original (`NEJM-Prod-Figures.R:23-27`)
```r
pacman::p_load(
  here, data.table, magrittr, janitor, IRdisplay, survival,
  lubridate, labelled, gt, tidyverse, knitr, survminer,
  gtsummary, GGally, broom
)
```

### Corrected
```r
# Core data manipulation
library(tidyverse)
library(data.table)
library(glue)

# Clinical analysis
library(survival)
library(survminer)
library(labelled)

# Tables and display
library(gt)
library(gtsummary)
library(knitr)

# Modeling
library(broom)

# Project infrastructure
library(here)
```

Note: `magrittr` is unnecessary when using `|>`. `IRdisplay` is a Jupyter
dependency — gate it with `if (requireNamespace("IRdisplay", quietly = TRUE))`.
`GGally` should only be loaded if actually used in the script.

---

## Summary of New Patterns Found

| # | Pattern | Files Affected | Severity |
|---|---------|---------------|----------|
| 1-3 | camelCase names (targets, variables, functions) | `_targets.R`, `functions.R` | High — propagates everywhere |
| 4 | `T`/`F` instead of `TRUE`/`FALSE` | `functions.R`, `functions-lhn.R` | High — can silently break |
| 5 | `TRUE ~` in `case_when()` | `functions.R`, `functions-lhn.R` | Medium — deprecated idiom |
| 6-7 | `stop()`/`stopifnot()` instead of `cli::cli_abort()` | `functions.R`, `functions-lhn.R` | Medium — poor error UX |
| 8 | `library()` mid-file | `functions.R` | Medium — hidden dependencies |
| 9 | `=` for assignment | `functions-lhn.R` | Low — style consistency |
| 10 | Explicit `return()` | `functions-lhn.R` | Low — style consistency |
| 11 | `cat()` for logging | `NEJM-Prod-Figures.R`, `functions-lhn.R` | Medium — unsuppressable |
| 12-13 | `sprintf()`/`ifelse()` | `NEJM-Prod-Figures.R` | Medium — readability, type safety |
| 14 | `map_df()` deprecated | `NEJM-Prod-Figures.R` | Medium — will warn in future |
| 15 | `size` deprecated in ggplot2 | `NEJM-Prod-Figures.R` | Medium — deprecation warning |
| 16 | Mixed quoting | `_targets.R` | Low — style consistency |
| 17 | `data.frame()` instead of `tibble()` | Multiple | Low — tidyverse consistency |
| 18 | `gsub()`/`paste0()` vs stringr/glue | `functions-lhn.R` | Low — readability |
| 19 | `cat()` debug logging | `functions-lhn.R` | Low — should use cli |
| 20 | `!!rlang::sym()` pattern | `functions-lhn.R` | Low — readability |
| 21 | camelCase column names in mutate | `functions.R` | High — propagates to all downstream |
| 22 | `pacman::p_load()` in prod scripts | `NEJM-Prod-Figures.R` | High — reproducibility |

### Note on `data.table` Usage in `functions-lhn.R`

The heavy use of `data.table` (`fread`, `dcast`, `setDT`, `setnames`, `.SD[1]`)
in `functions-lhn.R` is a **deliberate performance choice** for large-scale
feature pivoting operations. Per `method-decision-tree.md` §8 ("When NOT to
use tidyverse"), `data.table` is appropriate when:

- Processing millions of rows in feature engineering
- Performance-critical inner loops
- Wide pivots with `dcast` that need memory efficiency

The recommendation is NOT to rewrite these to tidyverse, but to:
1. Keep `data.table` usage contained to performance-critical functions
2. Accept `data.table` input/output at function boundaries
3. Convert to tibble at the boundary when passing to tidyverse pipelines
4. Document the reason for `data.table` choice in the function's roxygen
