# R Anti-Patterns — What to Avoid

**Authority:** Jenny Bryan's [Project-oriented workflow](https://tidyverse.org/blog/2017/12/workflow-vs-script/), [What They Forgot to Teach You About R](https://rstats.wtf/), and tidyverse team conventions.

---

## 1. Workflow Anti-Patterns

### NEVER: `setwd()`
```r
# Bad — breaks for everyone else
setwd("C:/Users/harlan/Documents/my_project")
read.csv("data.csv")

# Good — works for anyone with the project
library(here)
read_csv(here("data", "data.csv"))
```
**Why:** Hardcodes your machine's directory structure. Zero chance of working for collaborators.

### NEVER: `rm(list = ls())`
```r
# Bad — false sense of clean slate
rm(list = ls())

# Good — restart R session (Ctrl+Shift+F10 in RStudio/Positron)
```
**Why:** Only removes user objects, not loaded packages, options, or connections. Creates hidden dependencies.

### NEVER: `source()` a file that sources other files
If you need a pipeline, use **targets** or numbered scripts.

### NEVER: Save/load `.RData`
```r
# Disable in RStudio: Tools > Global Options > uncheck "Restore .RData"
# Or in .Rprofile:
# usethis::use_blank_slate()
```
**Why:** Hidden state. You can't tell what's in your environment by reading your scripts.

---

## 2. Coding Anti-Patterns

### Avoid: `T` and `F`
```r
# Bad — T and F can be overwritten
T <- FALSE  # This works! Now T is FALSE.
mean(x, na.rm = T)

# Good — TRUE and FALSE are reserved words
mean(x, na.rm = TRUE)
```

### Avoid: `=` for assignment
```r
# Bad
x = 10

# Good
x <- 10
```

### Avoid: `1:length(x)`
```r
# Bad — fails when x is empty (produces 1:0 = c(1, 0))
for (i in 1:length(x)) { ... }

# Good
for (i in seq_along(x)) { ... }
# Or better: use purrr::map()
```

### Avoid: `sapply()` (type-unstable)
```r
# Bad — return type depends on input values
sapply(list(1, 1:2), identity)

# Good — explicit type
vapply(list(1, 1:2), identity, double(1))
# Or better
map_dbl(1:3, sqrt)
```

### Avoid: `library()` inside functions
```r
# Bad — side effect that modifies search path
my_func <- function(x) {
  library(dplyr)
  x |> filter(a > 1)
}

# Good — use namespace qualification or declare in DESCRIPTION
my_func <- function(x) {
  dplyr::filter(x, a > 1)
}
```

### Avoid: `require()` in scripts
```r
# Bad — returns FALSE silently on failure
require(nonexistent_pkg)

# Good — fails immediately with clear error
library(nonexistent_pkg)
```

### Avoid: `attach()`
```r
# Bad — pollutes search path, creates name conflicts
attach(mtcars)
mean(mpg)

# Good — explicit reference
mean(mtcars$mpg)
# Or in a pipe
mtcars |> pull(mpg) |> mean()
```

---

## 3. Tidyverse-Specific Anti-Patterns

### Avoid: `gather()`/`spread()` (superseded)
```r
# Old
gather(data, key, value, -id)
spread(data, key, value)

# Current
pivot_longer(data, cols = -id, names_to = "key", values_to = "value")
pivot_wider(data, names_from = key, values_from = value)
```

### Avoid: `do()` (deprecated)
```r
# Old
group_by(cyl) |> do(model = lm(mpg ~ wt, data = .))

# Current
nest() |> mutate(model = map(data, \(df) lm(mpg ~ wt, data = df)))
```

### Avoid: `summarise_each()`/`mutate_each()` (deprecated)
```r
# Old
summarise_each(funs(mean))

# Current
summarize(across(everything(), mean))
```

### Avoid: `aes_string()` (deprecated)
```r
# Old
aes_string(x = "var1", y = "var2")

# Current — use .data pronoun
aes(x = .data[["var1"]], y = .data[["var2"]])
# Or tidy eval
aes(x = {{ var1 }}, y = {{ var2 }})
```

### Avoid: `%>%` when `|>` works
```r
# Prefer native pipe
data |> filter(x > 0) |> mutate(y = log(x))

# Only use magrittr when native pipe can't do it
data %>% lm(y ~ x, data = .)
```

---

## 4. Performance Anti-Patterns

### Avoid: Growing objects in loops
```r
# Bad — O(n²) due to copying
result <- c()
for (i in 1:n) {
  result <- c(result, compute(i))
}

# Good — pre-allocate
result <- vector("double", n)
for (i in seq_len(n)) {
  result[i] <- compute(i)
}

# Best — use purrr
result <- map_dbl(seq_len(n), compute)
```

### Avoid: `rbind()` in loops
```r
# Bad — copies entire data frame each iteration
df <- data.frame()
for (f in files) {
  df <- rbind(df, read_csv(f))
}

# Good
df <- map(files, read_csv) |> list_rbind()
```

### Avoid: `ifelse()` for type-stable results
```r
# Bad — ifelse() can coerce types unexpectedly
ifelse(condition, as.Date("2024-01-01"), as.Date("2024-06-01"))
# Returns numeric, not Date!

# Good — use dplyr::if_else() (type-stable)
if_else(condition, as.Date("2024-01-01"), as.Date("2024-06-01"))

# Good — use dplyr::case_when() for multiple conditions
case_when(
  x > 10 ~ "high",
  x > 5  ~ "medium",
  .default = "low"
)
```

---

## 5. Documentation Anti-Patterns

### Avoid: Comments that restate the code
```r
# Bad
# Add 1 to x
x <- x + 1

# Good — explain WHY
# Shift to 1-based indexing for compatibility with lookup table
x <- x + 1
```

### Avoid: Commented-out code
Delete it. It's in version control.

### Avoid: No documentation at all
At minimum: what does this script do? What does it expect as input?
