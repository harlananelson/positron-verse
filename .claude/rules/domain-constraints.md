# Domain Constraints — R Ecosystem

---

## 1. Language Version Constraints

| Constraint | Level | Notes |
|-----------|-------|-------|
| R ≥ 4.1 required for native pipe `\|>` | established | Released June 2021 |
| R ≥ 4.2 for `_` placeholder in native pipe | established | Released April 2022 |
| R ≥ 4.3 for native pipe lambda `\(x) x` everywhere | established | |
| CRAN packages must pass `R CMD check` | established | Zero errors, warnings, notes |
| UTF-8 is the default encoding | established | Since R 4.2 |

## 2. Tidyverse Version Constraints

| Constraint | Level | Notes |
|-----------|-------|-------|
| tidyverse 2.0+ uses `|>` in documentation | stable | As of 2023 |
| dplyr 1.1+ has `.by` argument (per-operation grouping) | stable | Preferred over `group_by()` for simple cases |
| ggplot2 3.5+ has improved theming | stable | |
| purrr 1.0+ deprecated formula syntax `~ .x` | stable | Use `\(x)` instead |
| tidyr 1.0+ replaced gather/spread with pivot_* | established | |
| rlang 1.0+ includes caller in error messages | established | |

## 3. Package Development Constraints

| Constraint | Level | Notes |
|-----------|-------|-------|
| DESCRIPTION must list all non-base dependencies | established | |
| Imports vs Suggests distinction matters | established | Imports = required; Suggests = optional |
| Use `@importFrom pkg fun` for selective imports | established | Avoid `@import pkg` |
| roxygen2 for documentation (not manual .Rd) | established | |
| testthat 3e (third edition) is current | stable | `Config/testthat/edition: 3` |
| pkgdown for package websites | stable | |

## 4. Project Workflow Constraints

| Constraint | Level | Notes |
|-----------|-------|-------|
| Use RStudio projects (.Rproj) or here package | established | Never setwd() |
| renv for dependency management | stable | Lockfile ensures reproducibility |
| targets for pipeline orchestration | stable | Replaces drake |
| Quarto for publishing (not R Markdown for new work) | stable | R Markdown still supported |
| Git for version control | established | |

## 5. Durability Levels

| Level | Meaning |
|-------|---------|
| **established** | Core R/tidyverse convention, unlikely to change |
| **stable** | Current best practice, maintained by Posit |
| **emerging** | New recommendation, may evolve |
| **contextual** | Depends on specific project requirements |
