# Output Conventions — R Analysis Products

---

## 1. Script Structure

Every R script should follow this structure:

```r
# Title: Brief description
# Author: Name
# Date: YYYY-MM-DD
# Purpose: What this script does and why

# Load packages ----
library(tidyverse)
library(here)

# Load data ----
data <- read_csv(here("data", "raw", "input.csv"))

# Transform ----
cleaned <- data |>
  filter(!is.na(key_var)) |>
  mutate(new_var = transform(old_var))

# Analyze ----
results <- cleaned |>
  group_by(group_var) |>
  summarize(metric = mean(value, na.rm = TRUE))

# Output ----
write_csv(results, here("data", "processed", "results.csv"))
```

---

## 2. Quarto/Notebook Structure

```yaml
---
title: "Descriptive Title"
author: "Name"
date: today
format:
  html:
    toc: true
    code-fold: true
    code-summary: "Show code"
---
```

### Section Order
1. Setup (packages, data loading)
2. Data overview / quality checks
3. Analysis / modeling
4. Results / visualizations
5. Conclusions / next steps

---

## 3. ggplot2 Output Standards

### Plot Structure
```r
p <- ggplot(data, aes(x = var1, y = var2)) +
  geom_point(alpha = 0.6) +
  labs(
    title = "Clear, Descriptive Title",
    subtitle = "Additional context if needed",
    x = "X Axis Label (units)",
    y = "Y Axis Label (units)",
    caption = "Source: data source"
  ) +
  theme_minimal(base_size = 12)
```

### Saving Plots
```r
ggsave(
  here("output", "figures", "descriptive-name.png"),
  plot = p,
  width = 8,
  height = 6,
  dpi = 300
)
```

---

## 4. Table Output Standards

### For Reports (gt)
```r
results |>
  gt() |>
  tab_header(title = "Table Title") |>
  fmt_number(columns = where(is.numeric), decimals = 2) |>
  cols_label(col1 = "Label 1", col2 = "Label 2")
```

### For Summary Statistics (gtsummary)
```r
data |>
  tbl_summary(
    by = group_var,
    include = c(var1, var2, var3),
    statistic = list(
      all_continuous() ~ "{mean} ({sd})",
      all_categorical() ~ "{n} ({p}%)"
    )
  ) |>
  add_p() |>
  bold_p()
```

---

## 5. Model Output Standards

### Always Report
- Model specification (engine, mode, hyperparameters)
- Resampling strategy (folds, strata)
- Primary metric with confidence interval
- Variable importance (top 10+)

### Standard Metrics by Task

| Task | Primary | Secondary |
|------|---------|-----------|
| Classification | ROC AUC | Brier score, accuracy, sensitivity, specificity |
| Regression | RMSE | R², MAE |
| Survival | C-index | Brier score, calibration |

---

## 6. Data Output Standards

### File Formats
- **CSV** for tabular data shared with others: `write_csv()`
- **Parquet** for large datasets or internal use: `arrow::write_parquet()`
- **RDS** for R-specific objects: `write_rds()` (preserves types)
- Never `.RData` / `.rda` for analysis outputs

### File Organization
```
project/
├── data/
│   ├── raw/           # Never modify originals
│   ├── processed/     # Cleaned, ready for analysis
│   └── external/      # Downloaded/API data
├── output/
│   ├── figures/       # Saved plots
│   ├── tables/        # Exported tables
│   └── models/        # Saved model objects
└── reports/           # Rendered Quarto output
```
