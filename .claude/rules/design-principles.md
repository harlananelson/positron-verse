# Tidy Design Principles

**Authority:** [Tidy Design Principles](https://design.tidyverse.org/) and [The Tidy Tools Manifesto](https://tidyverse.tidyverse.org/articles/manifesto.html)

---

## 1. The Four Pillars (Tidy Tools Manifesto)

### 1.1 Reuse Existing Data Structures
- Use data frames/tibbles as the primary data structure
- When custom structures are needed, build S3 classes on atomic vectors or lists
- Prioritize compatibility over perfect fit
- Never invent a new container when a tibble works

### 1.2 Compose Simple Functions with the Pipe
- Each function does ONE thing well
- Functions should be composable via `|>`
- Avoid mixing side-effects with transformations
- Name functions as verbs (with nouns when the verb repeats):
  ```r
  # Good: verb-based, composable
  filter() |> select() |> mutate() |> summarize()

  # Bad: monolithic
  filter_select_mutate_summarize()
  ```

### 1.3 Embrace Functional Programming
- Immutable objects, copy-on-modify
- Use `purrr::map()` family over explicit loops
- S3 generics for polymorphism
- Functions as first-class objects

### 1.4 Design for Humans
- Invest in evocative function and argument names
- Prefer explicit over abbreviated
- Use consistent prefixes for function families
- "Programs must be written for people to read" — Hal Abelson

---

## 2. Type Stability

**Rule:** The output type of a function should be predictable from its input types alone, not from input values.

```r
# Type-stable: map_dbl always returns double
map_dbl(1:3, sqrt)

# Type-unstable: sapply return type depends on input
sapply(list(1, 1:2), identity)  # sometimes vector, sometimes list
```

### Guidelines
- All purrr functions are type-stable (map → list, map_dbl → double, etc.)
- Prefer `vapply()` over `sapply()` when not using purrr
- Use vctrs coercion rules for custom type handling
- If a function could return different types, make separate functions

---

## 3. Data-First Convention

**Rule:** Data arguments come first to enable piping.

```r
# Good: data first
mutate(.data, new_col = old_col + 1)

# Pattern: data → ... → details
my_function <- function(data, ..., na.rm = FALSE) {
  # data is first, optional args via ..., details last
}
```

### Argument Ordering
1. Data arguments (enables piping)
2. `...` (flexible inputs)
3. Detail/option arguments (with defaults)

---

## 4. Tidy Data Principles

**Authority:** Hadley Wickham, "Tidy Data" (2014) and [R4DS Chapter: Tidy Data](https://r4ds.hadley.nz/data-tidy)

### The Three Rules
1. Each variable is a column; each column is a variable
2. Each observation is a row; each row is an observation
3. Each value is a cell; each cell is a single value

### In Practice
```r
# Tidy: each row is one measurement
tibble(
  country = c("A", "A", "B", "B"),
  year    = c(2020, 2021, 2020, 2021),
  cases   = c(100, 120, 200, 180)
)

# Untidy: years as columns
tibble(
  country = c("A", "B"),
  `2020`  = c(100, 200),
  `2021`  = c(120, 180)
)

# Fix with pivot_longer()
untidy |>
  pivot_longer(cols = c(`2020`, `2021`), names_to = "year", values_to = "cases")
```

---

## 5. Tidy Selection

Use tidy selection helpers consistently:

```r
# By type
across(where(is.numeric), mean)

# By name pattern
across(starts_with("score_"), scale)
across(ends_with("_date"), as.Date)
across(contains("temp"), \(x) x * 9/5 + 32)
across(matches("^x[0-9]+$"), log)

# By position (rare)
across(1:5, as.character)

# Combining
across(c(where(is.numeric), -id), scale)
```

---

## 6. Tidy Evaluation (for Function Authors)

### The Embrace Operator `{{ }}`
```r
# Wrapping dplyr verbs in custom functions
my_summary <- function(data, group_var, value_var) {
  data |>
    group_by({{ group_var }}) |>
    summarize(
      mean = mean({{ value_var }}, na.rm = TRUE),
      n = n()
    )
}

# Usage: columns passed as bare names
my_summary(mtcars, cyl, mpg)
```

### The .data and .env Pronouns
```r
# .data refers to data frame columns (avoids ambiguity)
filter(df, .data[["column_name"]] > 5)

# .env refers to environment variables
threshold <- 5
filter(df, .data$value > .env$threshold)
```

### String-to-Symbol
```r
# When column names come as strings
col_name <- "mpg"
mtcars |> select(all_of(col_name))
mtcars |> filter(.data[[col_name]] > 20)
```

---

## 7. Vector Recycling (vctrs rules)

- Size 1 vectors recycle to match any other size
- Non-size-1 vectors must all be the same size
- This is stricter than base R (which silently recycles any multiple)

```r
# OK: scalar recycles
tibble(x = 1:3, y = 1)

# Error in tidyverse: incompatible sizes
tibble(x = 1:3, y = 1:2)  # Error!
```

---

## 8. Error Handling (rlang patterns)

```r
# Use cli-formatted error messages
my_function <- function(x, arg = caller_arg(x), call = caller_env()) {
  if (!is.numeric(x)) {
    cli::cli_abort(
      "{.arg {arg}} must be numeric, not {.obj_type_friendly {x}}.",
      call = call
    )
  }
}

# Three levels of communication
cli::cli_abort("...")    # Error: stop execution
cli::cli_warn("...")     # Warning: continue but notify
cli::cli_inform("...")   # Message: informational
```

---

## 9. Naming Conventions for Custom Packages

- Package name: lowercase, no dots or underscores
- Functions: `verb_noun()` pattern (`read_csv()`, `add_recipe()`)
- Classes: PascalCase for R6, lowercase for S3
- Use consistent prefixes for function families
- Accessor functions: `get_*()` or just the noun
- Predicate functions: `is_*()`, `has_*()`
