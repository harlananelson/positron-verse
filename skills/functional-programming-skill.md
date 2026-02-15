# Functional Programming in R Skill

> **Usage:** Load this file into Claude Code (via CLAUDE.md pointer) or paste into an LLM session for writing functional R code following Posit conventions.

Comprehensive reference for functional programming in R with purrr, rlang, and base R FP tools. Based on *Advanced R (2e)* (https://adv-r.hadley.nz/) chapters on functions, functionals, and function factories.

---

## 1) Core Principles

### Functional Programming in R Means

1. **Functions are first-class objects** — pass them to other functions, store in lists
2. **Immutable data** — copy-on-modify semantics; transformations return new objects
3. **Declarative style** — say *what* to do, not *how* to iterate
4. **Composition via pipes** — chain small functions into expressive workflows

### When to Use FP vs Loops

| Use purrr/FP | Use for loop |
|-------------|------------|
| Apply same operation to many elements | Complex multi-step iteration with side effects |
| Type-stable output needed | Iterative algorithms (convergence, etc.) |
| Working in a pipe | When clarity demands explicit state tracking |
| Need error handling per element | Performance-critical tight loops (or use Rcpp) |

---

## 2) The map() Family

### Basic Variants
```r
map(x, fun)         # → list
map_chr(x, fun)     # → character vector
map_dbl(x, fun)     # → double vector
map_int(x, fun)     # → integer vector
map_lgl(x, fun)     # → logical vector
map_vec(x, fun)     # → auto-simplified (vctrs rules)
```

### Multi-Input
```r
map2(x, y, \(a, b) a + b)               # Two inputs, parallel
pmap(list(a = x, b = y, c = z), fun)     # Multiple inputs
imap(x, \(value, index) paste(index, value))  # With names/indices
```

### Side Effects
```r
walk(paths, \(p) write_csv(data, p))         # Like map, returns input invisibly
walk2(data_list, paths, \(d, p) write_csv(d, p))
iwalk(results, \(val, name) cat(name, ": ", val, "\n"))
```

### List Operations
```r
list_rbind(list_of_dfs)         # Bind rows
list_cbind(list_of_dfs)         # Bind columns
list_c(list_of_vectors)         # Concatenate
list_flatten(nested_list)       # Remove one level of nesting
```

---

## 3) Lambda (Anonymous Function) Syntax

### Modern R (preferred)
```r
# Short, one-line
map_dbl(x, \(i) i^2)

# Multi-line
map(data_list, \(df) {
  df |>
    filter(!is.na(value)) |>
    summarize(mean = mean(value))
})
```

### Legacy purrr Formula (avoid in new code)
```r
# Still works but discouraged since purrr 1.0
map_dbl(x, ~ .x^2)
map2(x, y, ~ .x + .y)
```

### Named Function (for complex or reusable logic)
```r
compute_summary <- function(df) {
  df |>
    filter(!is.na(value)) |>
    summarize(mean = mean(value), n = n())
}

map(data_list, compute_summary)
```

---

## 4) Error Handling in Functional Style

### safely() — Capture Errors
```r
safe_log <- safely(log, otherwise = NA_real_)
results <- map(x, safe_log)

# Extract successes and failures
map(results, "result")  # List of results (NA where failed)
map(results, "error")   # List of errors (NULL where succeeded)
```

### possibly() — Default on Error
```r
map_dbl(x, possibly(log, otherwise = NA_real_))
```

### quietly() — Capture Warnings and Messages
```r
quiet_fn <- quietly(function(x) {
  if (x < 0) warning("Negative input")
  sqrt(abs(x))
})
```

### insistently() — Retry on Failure
```r
# Retry up to 3 times with exponential backoff
resilient_fetch <- insistently(fetch_data, rate = rate_backoff(max_times = 3))
map(urls, resilient_fetch)
```

---

## 5) Predicate Functions

```r
keep(x, is.numeric)           # Keep elements where predicate is TRUE
discard(x, is.na)             # Remove elements where predicate is TRUE
detect(x, \(i) i > 10)        # First element matching predicate
detect_index(x, \(i) i > 10)  # Index of first match
every(x, is.numeric)          # All elements match?
some(x, is.na)                # Any element matches?
none(x, is.na)                # No elements match?
```

---

## 6) Reduce and Accumulate

```r
# Reduce list to single value
reduce(list(df1, df2, df3), left_join, by = "id")
reduce(1:5, `+`)  # 15

# Accumulate (reduce with intermediate results)
accumulate(1:5, `+`)  # 1, 3, 6, 10, 15
```

---

## 7) Deep Extraction with pluck()

```r
# Safe deep extraction (never errors)
pluck(x, "nested", "field", 1)
pluck(x, "missing", .default = "fallback")

# Equivalent to:
# x[["nested"]][["field"]][[1]]
# But safe — returns NULL (or .default) on missing

# Set values
pluck(x, "nested", "field") <- new_value
```

---

## 8) Function Factories

```r
# Function that returns a function
power <- function(exp) {
  \(x) x^exp
}

square <- power(2)
cube <- power(3)
square(4)  # 16
cube(4)    # 64

# Practical: custom summary
make_summarizer <- function(fun, na.rm = TRUE) {
  \(x) fun(x, na.rm = na.rm)
}

safe_mean <- make_summarizer(mean)
safe_sd <- make_summarizer(sd)
```

---

## 9) Tidy Evaluation for Function Authors

### The Embrace Operator `{{ }}`
```r
# When writing functions that wrap dplyr/tidyr
grouped_mean <- function(data, group_var, value_var) {
  data |>
    summarize(mean = mean({{ value_var }}, na.rm = TRUE), .by = {{ group_var }})
}

# Usage
mtcars |> grouped_mean(cyl, mpg)
```

### Passing `...` Through
```r
my_select <- function(data, ...) {
  data |> select(...)
}
```

### String-to-Column
```r
# When column name comes as a string
filter_by <- function(data, col_name, value) {
  data |> filter(.data[[col_name]] == value)
}

# When column names come as character vector
select_cols <- function(data, cols) {
  data |> select(all_of(cols))
}
```

---

## 10) Common Functional Patterns

### Split-Apply-Combine
```r
# Modern tidyverse way
data |>
  nest(.by = group) |>
  mutate(result = map(data, analyze)) |>
  unnest(result)

# Or with group_map
data |>
  group_by(group) |>
  group_map(\(df, key) analyze(df))
```

### Read and Combine Multiple Files
```r
paths <- list.files(here("data"), pattern = "\\.csv$", full.names = TRUE)

# Simple
all_data <- map(paths, read_csv) |> list_rbind()

# With file info
all_data <- map(paths, \(p) {
  read_csv(p) |>
    mutate(source = basename(p))
}) |> list_rbind()

# With progress
all_data <- map(paths, read_csv, .progress = TRUE) |> list_rbind()
```

### Many Models
```r
models <- mtcars |>
  nest(.by = cyl) |>
  mutate(
    model = map(data, \(df) lm(mpg ~ wt + hp, data = df)),
    tidy  = map(model, broom::tidy),
    glance = map(model, broom::glance)
  )

# Extract all coefficients
models |> unnest(tidy)

# Extract all model summaries
models |> unnest(glance)
```

---

## 11) Key References

- **Advanced R — Functionals**: https://adv-r.hadley.nz/functionals.html
- **Advanced R — Function factories**: https://adv-r.hadley.nz/function-factories.html
- **purrr reference**: https://purrr.tidyverse.org/
- **rlang tidy eval**: https://rlang.r-lib.org/reference/topic-data-mask.html
- **Programming with dplyr**: https://dplyr.tidyverse.org/articles/programming.html
