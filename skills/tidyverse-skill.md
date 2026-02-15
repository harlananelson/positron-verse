# Tidyverse Core Skill

> **Usage:** Load this file into Claude Code (via CLAUDE.md pointer) or paste into an LLM session for writing idiomatic tidyverse R code following Posit conventions.

Comprehensive reference for the core tidyverse packages: dplyr, tidyr, purrr, readr, tibble, stringr, forcats, and lubridate. Covers the patterns promoted by Hadley Wickham, Jenny Bryan, and the Posit team.

---

## 1) dplyr — Data Manipulation Grammar

### Core Verbs
```r
data |>
  filter(condition)            # Keep rows matching condition
  select(col1, col2)           # Keep/reorder columns
  mutate(new = transform(old)) # Create/modify columns
  summarize(stat = fun(col))   # Reduce rows to summaries
  arrange(col)                 # Sort rows
  rename(new = old)            # Rename columns
  relocate(col, .after = x)   # Move columns
  distinct(col)                # Unique rows
  count(col)                   # Frequency table
  pull(col)                    # Extract single column as vector
  slice_head(n = 5)            # First n rows
  slice_sample(n = 10)         # Random rows
```

### Grouping
```r
# Per-operation grouping (preferred for simple cases, dplyr 1.1+)
data |>
  summarize(mean_x = mean(x), .by = group_col)

# Persistent grouping (for multiple operations on same groups)
data |>
  group_by(group_col) |>
  mutate(centered = x - mean(x)) |>
  summarize(n = n()) |>
  ungroup()  # Always ungroup when done
```

### across() — Apply Functions to Multiple Columns
```r
data |>
  summarize(across(where(is.numeric), list(mean = mean, sd = sd), na.rm = TRUE))

data |>
  mutate(across(starts_with("score"), \(x) (x - mean(x)) / sd(x)))

data |>
  filter(if_any(c(col1, col2), \(x) !is.na(x)))
```

### Joins
```r
left_join(x, y, by = "key")                    # Keep all rows from x
left_join(x, y, by = join_by(x_key == y_key))  # Different column names
inner_join(x, y, by = "key")                   # Only matching rows
anti_join(x, y, by = "key")                    # Rows in x not in y
semi_join(x, y, by = "key")                    # Rows in x that match y
```

### Conditional Logic
```r
# Two outcomes
if_else(condition, true_value, false_value)  # Type-stable (NOT base ifelse)

# Multiple outcomes
case_when(
  x > 100 ~ "high",
  x > 50  ~ "medium",
  .default = "low"
)

# Match specific values
case_match(
  status,
  "A" ~ "Active",
  c("I", "X") ~ "Inactive",
  .default = "Unknown"
)
```

---

## 2) tidyr — Reshaping and Tidying

### Pivoting
```r
# Wide to long
data |>
  pivot_longer(
    cols = starts_with("year_"),
    names_to = "year",
    names_prefix = "year_",
    values_to = "value"
  )

# Long to wide
data |>
  pivot_wider(
    names_from = category,
    values_from = value,
    values_fill = 0
  )
```

### Nesting and Unnesting
```r
# Nest data by group
nested <- data |>
  nest(.by = group_col)

# Apply models to each group
nested |>
  mutate(
    model = map(data, \(df) lm(y ~ x, data = df)),
    tidy = map(model, broom::tidy)
  ) |>
  unnest(tidy)
```

### Separating and Uniting
```r
# Split one column into many
data |> separate_wider_delim(col, delim = "-", names = c("a", "b", "c"))
data |> separate_wider_regex(col, patterns = c(code = "[A-Z]+", num = "[0-9]+"))

# Combine columns
data |> unite("full_name", first, last, sep = " ")
```

### Missing Values
```r
data |> drop_na(col1, col2)              # Remove rows with NA in specified cols
data |> fill(col, .direction = "down")   # Fill NA with previous value
data |> replace_na(list(col = 0))        # Replace NA with specific value
```

---

## 3) purrr — Functional Programming

### map() Family (type-stable)
```r
map(x, fun)         # Always returns a list
map_chr(x, fun)     # Always returns character vector
map_dbl(x, fun)     # Always returns double vector
map_int(x, fun)     # Always returns integer vector
map_lgl(x, fun)     # Always returns logical vector
map_vec(x, fun)     # Auto-simplifies (vctrs rules)
```

### Two-Input Iteration
```r
map2(x, y, \(a, b) a + b)          # Iterate over two inputs
map2_dbl(x, y, \(a, b) a + b)     # Type-stable variant
pmap(list(a, b, c), \(a, b, c) a + b + c)  # Multiple inputs
```

### Side Effects
```r
walk(files, \(f) write_csv(data, f))   # Like map but returns input invisibly
iwalk(x, \(val, name) cat(name, ":", val, "\n"))  # With index/name
```

### List Manipulation
```r
list_rbind(list_of_dfs)    # Bind list of data frames by rows
list_c(list_of_vectors)    # Concatenate list of vectors
keep(x, is.numeric)        # Keep elements matching predicate
discard(x, is.na)          # Remove elements matching predicate
pluck(x, "nested", 1)     # Deep extraction
```

### Lambda Syntax
```r
# Modern (preferred)
map(x, \(item) item$value + 1)

# Avoid legacy formula syntax
map(x, ~ .x$value + 1)  # Still works but discouraged in purrr 1.0+
```

### Safely Handling Errors
```r
safe_log <- safely(log)
results <- map(x, safe_log)
successes <- map(results, "result") |> compact()
errors <- map(results, "error") |> compact()

# Or with possibly() for default on error
map_dbl(x, possibly(log, otherwise = NA_real_))
```

---

## 4) readr — Data Import

```r
# CSV (most common)
data <- read_csv(here("data", "file.csv"))
data <- read_csv(here("data", "file.csv"),
  col_types = cols(
    id = col_character(),
    date = col_date(format = "%Y-%m-%d"),
    value = col_double()
  )
)

# TSV
data <- read_tsv(here("data", "file.tsv"))

# Fixed width
data <- read_fwf(here("data", "file.txt"),
  fwf_widths(c(10, 5, 8), c("name", "age", "score"))
)

# Write
write_csv(data, here("output", "result.csv"))
```

---

## 5) stringr — String Manipulation

```r
str_detect(x, "pattern")       # TRUE/FALSE for matches
str_count(x, "pattern")        # Number of matches
str_extract(x, "pattern")      # First match
str_extract_all(x, "pattern")  # All matches
str_replace(x, "old", "new")   # Replace first match
str_replace_all(x, "old", "new")  # Replace all
str_remove(x, "pattern")       # Remove first match
str_to_lower(x)                # Lowercase
str_to_title(x)                # Title Case
str_trim(x)                    # Remove whitespace
str_pad(x, width = 5, pad = "0")  # Pad strings
str_glue("Hello {name}")       # String interpolation
str_c("a", "b", sep = "-")    # Concatenate
str_split(x, ",")              # Split string
```

---

## 6) forcats — Factor Handling

```r
fct_reorder(f, x, .fun = median)  # Reorder by another variable
fct_infreq(f)                      # Order by frequency
fct_rev(f)                         # Reverse order
fct_relevel(f, "ref_level")       # Move specific level first
fct_lump_n(f, n = 5)              # Keep top n, lump rest to "Other"
fct_recode(f, "New" = "old")      # Rename levels
fct_collapse(f, group = c("a", "b"))  # Combine levels
fct_explicit_na(f)                 # Make NA a level
```

---

## 7) lubridate — Dates and Times

```r
# Parsing
ymd("2024-01-15")
mdy("01/15/2024")
dmy("15-Jan-2024")
ymd_hms("2024-01-15 10:30:00")

# Extraction
year(date)
month(date, label = TRUE)
day(date)
wday(date, label = TRUE)
quarter(date)

# Arithmetic
date + days(30)
date + months(3)
interval(start, end) / years(1)  # Duration in years
difftime(end, start, units = "days")

# Rounding
floor_date(date, unit = "month")
ceiling_date(date, unit = "week")
```

---

## 8) Common Patterns

### Read Multiple Files
```r
paths <- list.files(here("data"), pattern = "\\.csv$", full.names = TRUE)
all_data <- map(paths, read_csv) |> list_rbind()

# With file name as column
all_data <- map(paths, \(path) {
  read_csv(path) |> mutate(source = basename(path))
}) |> list_rbind()
```

### Pipe-Friendly Custom Functions
```r
# Data-first, return data for chaining
add_bmi <- function(data, height_col = height, weight_col = weight) {
  data |>
    mutate(bmi = {{ weight_col }} / ({{ height_col }} / 100)^2)
}

# Usage in pipe
patients |>
  add_bmi() |>
  filter(bmi > 25)
```

### Exploratory Data Analysis Pattern
```r
data |> glimpse()                          # Structure overview
data |> count(category, sort = TRUE)       # Frequency tables
data |> summarize(across(where(is.numeric), list(
  mean = \(x) mean(x, na.rm = TRUE),
  sd = \(x) sd(x, na.rm = TRUE),
  na = \(x) sum(is.na(x))
)))
```
