# Tidyverse Style Conventions

**Authority:** [Tidyverse Style Guide](https://style.tidyverse.org/) — the official coding standard for all tidyverse and Posit-promoted R code.

---

## 1. File Organization

### File Naming
- All lowercase, no spaces or special characters
- Separate words with hyphens or underscores
- Use `.R` extension (uppercase R)
- ISO8601 dates: `2024-01-15-analysis.R`
- Pad numbers: `01-load-data.R`, `02-clean.R`, `03-model.R`
- Never use "final" in filenames

### File Structure
- Load ALL packages at the top with `library()`
- Use commented separator lines to divide sections:
  ```r
  # Load data ----
  # Clean data ----
  # Model ----
  ```

---

## 2. Naming Conventions

### Variables and Functions
- **snake_case** only: `day_one`, `short_flights`
- Variables are **nouns**: `flight_data`, `model_fit`
- Functions are **verbs**: `add_row()`, `compute_metrics()`
- Descriptive over brief: `day_one` not `d1`
- Use common prefixes for related functions (aids autocomplete):
  ```r
  # Good: common prefix
  perm_check()
  perm_set()
  perm_list()

  # Bad: common suffix
  check_perm()
  set_perm()
  list_perm()
  ```

### Boolean Variables
- Use `is_`, `has_`, `can_` prefixes: `is_valid`, `has_header`

### Constants
- SCREAMING_SNAKE_CASE: `MAX_ITERATIONS`, `DEFAULT_ALPHA`

---

## 3. Syntax

### Assignment
```r
# Good
x <- 10

# Bad
x = 10
x <<- 10  # avoid global assignment
```

### Spacing
```r
# Spaces around operators (except ^)
height <- (feet * 12) + inches
sqrt(x^2 + y^2)
x <- 1:10

# Space after comma, not before
x[, 1]
mean(x, na.rm = TRUE)

# No spaces inside/outside parentheses for calls
mean(x, na.rm = TRUE)  # Good
mean( x, na.rm = TRUE) # Bad
```

### Indentation
- Two spaces per level (never tabs)
- Continuation lines aligned or indented by two spaces

### Line Length
- Aim for 80 characters, hard limit at 120

### Logical Values
```r
# Good
TRUE
FALSE

# Bad
T
F
```

---

## 4. Pipe Conventions

### Use Native Pipe `|>`
```r
# Good — native pipe (R >= 4.1)
flights |>
  filter(!is.na(arr_delay)) |>
  group_by(dest) |>
  summarize(avg_delay = mean(arr_delay))

# Acceptable — magrittr pipe (only when you need its features)
flights %>%
  mutate(., across(where(is.numeric), scale))
```

### When magrittr `%>%` Is Still Needed
- Using `.` placeholder in non-first-argument position (R 4.2+ has `_` but with limitations)
- Complex lambda expressions: `{...}` blocks
- `%T>%` (tee pipe) or `%$%` (exposition pipe)

### Formatting Rules
```r
# Space before pipe, newline after
flights |>
  filter(dep_delay > 60) |>
  group_by(carrier) |>
  summarize(n = n())

# Short pipes can be one line
x |> log() |> round(2)

# Break pipes longer than ~10 steps into named sub-tasks
```

### Assignment with Pipes
```r
# Preferred — assignment on first line
delayed_flights <- flights |>
  filter(dep_delay > 60)

# Also acceptable — assignment on separate line
delayed_flights <-
  flights |>
  filter(dep_delay > 60)
```

---

## 5. Function Conventions

### Anonymous Functions
```r
# Good — modern lambda syntax
map(x, \(i) i + 1)

# Good — for multi-line
map(x, \(i) {
  result <- i + 1
  result * 2
})

# Avoid — old-style formula (purrr legacy)
map(x, ~ .x + 1)
```

### Return Values
```r
# Implicit return (preferred for final value)
add_one <- function(x) {
  x + 1
}

# Explicit return (only for early exits)
safe_log <- function(x) {
  if (x <= 0) {
    return(NA_real_)
  }
  log(x)
}
```

### Side-Effect Functions
```r
# Return first argument invisibly for pipe chaining
print_summary <- function(data) {
  cat("Rows:", nrow(data), "\n")
  invisible(data)
}
```

### Comments in Functions
```r
# Explain WHY, not WHAT
# Cap outliers to prevent model instability from extreme leverage points
x[x > threshold] <- threshold
```

---

## 6. ggplot2 Style

```r
# One layer per line, + at end of line
ggplot(data, aes(x = var1, y = var2)) +
  geom_point(alpha = 0.5) +
  geom_smooth(method = "lm") +
  scale_x_log10() +
  labs(
    title = "Descriptive Title",
    x = "X Label",
    y = "Y Label"
  ) +
  theme_minimal()
```

---

## 7. Style Enforcement

### Automated Tools
- **styler** package: `styler::style_file("script.R")` or RStudio addin
- **lintr** package: `lintr::lint("script.R")` for static analysis
- Both default to tidyverse style

### Pre-commit
```r
# In package development
usethis::use_tidy_style()  # Applies styler to package
```
