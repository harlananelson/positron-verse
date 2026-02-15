# Positron-Verse Corrections — R for Data Science (1st Edition)

> **Source:** https://r4ds.had.co.nz/ (Wickham & Grolemund, 2017)
>
> R4DS 1e is a landmark book that established tidyverse conventions. However,
> it predates R 4.1 (native pipe), purrr 1.0 (deprecated functions), dplyr 1.1
> (.by grouping, .default in case_when), and ggplot2 3.4 (linewidth). The 2nd
> edition (https://r4ds.hadley.nz/) corrects most of these, but 1e remains
> widely referenced.
>
> This document shows what would change if R4DS 1e code were written today
> under positron-verse conventions.

---

## 1. `%>%` → `|>` (throughout the entire book)

**Rule:** `CLAUDE.md` invariant #1 — Use the native pipe `|>`, not `%>%`.
The native pipe (R 4.1+) has no package dependency and is now the standard.

### Original (Ch. 5 — Data Transformation)
```r
delays <- flights %>%
  group_by(dest) %>%
  summarise(
    count = n(),
    dist = mean(distance, na.rm = TRUE),
    delay = mean(arr_delay, na.rm = TRUE)
  ) %>%
  filter(count > 20, dest != "HNL")
```

### Corrected
```r
delays <- flights |>
  group_by(dest) |>
  summarise(
    count = n(),
    dist = mean(distance, na.rm = TRUE),
    delay = mean(arr_delay, na.rm = TRUE)
  ) |>
  filter(count > 20, dest != "HNL")
```

**Scope:** This applies to every single pipe chain in the book — hundreds of
occurrences across all chapters. The `magrittr` package is no longer needed.

---

## 2. `group_by() |> summarise()` → `.by` argument

**Rule:** `tidyverse-skill.md` §1.2 — For single-operation grouping, prefer
the `.by` argument (dplyr 1.1.0+) which automatically ungroups. Reserve
`group_by()` for multi-step grouped operations.

### Original (Ch. 5)
```r
not_cancelled %>%
  group_by(year, month, day) %>%
  summarise(mean = mean(dep_delay, na.rm = TRUE))
```

### Corrected
```r
not_cancelled |>
  summarise(
    mean = mean(dep_delay, na.rm = TRUE),
    .by = c(year, month, day)
  )
```

### Original (Ch. 5 — count)
```r
not_cancelled %>%
  group_by(dest) %>%
  summarise(carriers = n_distinct(carrier)) %>%
  arrange(desc(carriers))
```

### Corrected
```r
not_cancelled |>
  summarise(
    carriers = n_distinct(carrier),
    .by = dest
  ) |>
  arrange(desc(carriers))
```

---

## 3. `~` formula lambdas → `\()` anonymous functions

**Rule:** `anti-patterns.md` §3.3 — Use `\(x)` lambda syntax (R 4.1+)
instead of purrr formula shorthand `~.x`. The native syntax is clearer
and doesn't require purrr.

### Original (Ch. 21 — Iteration)
```r
models <- mtcars %>%
  split(.$cyl) %>%
  map(~lm(mpg ~ wt, data = .))

models %>%
  map(summary) %>%
  map_dbl(~.$r.squared)
```

### Corrected
```r
models <- mtcars |>
  split(mtcars$cyl) |>
  map(\(df) lm(mpg ~ wt, data = df))

models |>
  map(summary) |>
  map_dbl(\(x) x$r.squared)
```

### Original (Ch. 25 — Many Models)
```r
df %>%
  mutate(
    smry = map2_chr(name, value, ~ stringr::str_c(.x, ": ", .y[1]))
  )
```

### Corrected
```r
df |>
  mutate(
    smry = map2_chr(name, value, \(nm, val) str_c(nm, ": ", val[1]))
  )
```

---

## 4. `.$` inside pipe → explicit pronoun or pre-pipe variable

**Rule:** `style-conventions.md` §4 — The `.` pronoun is a magrittr
feature. With `|>`, use the data directly or name the variable.

### Original (Ch. 21)
```r
models <- mtcars %>%
  split(.$cyl) %>%
  map(~lm(mpg ~ wt, data = .))
```

### Corrected
```r
models <- mtcars |>
  split(mtcars$cyl) |>
  map(\(df) lm(mpg ~ wt, data = df))
```

Note: The native pipe doesn't support `.` as a placeholder. Either use
`\(x) split(x, x$cyl)` or reference the variable name directly.

---

## 5. `gather()` / `spread()` → `pivot_longer()` / `pivot_wider()`

**Rule:** `anti-patterns.md` §3.1 — `gather()` and `spread()` were
superseded by `pivot_longer()` and `pivot_wider()` in tidyr 1.0.0 (2019).
The new functions have clearer names and more flexible syntax.

The R4DS 1e tidy data chapter was already updated to use `pivot_longer()`
and `pivot_wider()`, so the main text is fine. But `gather_predictions()`
and `gather_residuals()` from modelr appear in the modeling chapters:

### Original (Ch. 23 — Model Basics)
```r
grid <- sim3 %>%
  data_grid(x1, x2) %>%
  gather_predictions(mod1, mod2)

sim3 <- sim3 %>%
  gather_residuals(mod1, mod2)
```

### Corrected
These modelr functions are acceptable since they produce a tidy output with
a `model` column — there's no tidyr equivalent. But for new code, consider
broom-based workflows:

```r
# Alternative using purrr + broom
models <- list(additive = mod1, interaction = mod2)

predictions <- models |>
  imap(\(mod, name) {
    grid |>
      add_predictions(mod) |>
      mutate(model = name)
  }) |>
  list_rbind()
```

---

## 6. `invoke_map()` (deprecated) → modern alternatives

**Rule:** `anti-patterns.md` §3.2 — `invoke_map()` was deprecated in
purrr 1.0.0. Use `map()` with `exec()` or `rlang::exec()`.

### Original (Ch. 21, Ch. 25)
```r
sim <- tribble(
  ~f,      ~params,
  "runif", list(min = -1, max = 1),
  "rnorm", list(sd = 5),
  "rpois", list(lambda = 10)
)

sim %>%
  mutate(sims = invoke_map(f, params, n = 10))
```

### Corrected
```r
sim <- tribble(
  ~f,      ~params,
  "runif", list(min = -1, max = 1),
  "rnorm", list(sd = 5),
  "rpois", list(lambda = 10)
)

sim |>
  mutate(sims = map2(f, params, \(fn, args) exec(fn, !!!args, n = 10)))
```

---

## 7. `sapply()` → `map_*()` or `vapply()`

**Rule:** `anti-patterns.md` §2.5 — `sapply()` silently simplifies output
type, which can cause unpredictable results. Use `map_lgl()`, `map_chr()`,
`map_dbl()`, etc. for type-stable output.

### Original (Ch. 21)
```r
x1 %>% sapply(threshold) %>% str()
x2 %>% sapply(threshold) %>% str()

# And later:
col_sum3 <- function(df, f) {
  is_num <- sapply(df, is.numeric)
  df_num <- df[, is_num]
  sapply(df_num, f)
}
```

### Corrected
```r
x1 |> map(threshold) |> str()
x2 |> map(threshold) |> str()

col_sum3 <- function(df, f) {
  is_num <- map_lgl(df, is.numeric)
  df_num <- df[, is_num]
  map_dbl(df_num, f)
}
```

The book itself notes that `sapply()` is unreliable — this correction
follows the book's own advice.

---

## 8. `stop()` → `cli::cli_abort()`

**Rule:** `design-principles.md` §8 — Use `cli::cli_abort()` for structured,
informative error messages.

### Original (Ch. 19 — Functions)
```r
wt_mean <- function(x, w) {
  if (length(x) != length(w)) {
    stop("`x` and `w` must be the same length", call. = FALSE)
  }
  sum(w * x) / sum(w)
}

wt_mean <- function(x, w, na.rm = FALSE) {
  if (!is.logical(na.rm)) {
    stop("`na.rm` must be logical")
  }
  if (length(na.rm) != 1) {
    stop("`na.rm` must be length 1")
  }
  if (length(x) != length(w)) {
    stop("`x` and `w` must be the same length", call. = FALSE)
  }
  ...
}
```

### Corrected
```r
wt_mean <- function(x, w) {
  if (length(x) != length(w)) {
    cli::cli_abort(
      "{.arg x} (length {length(x)}) and {.arg w} (length {length(w)}) must be the same length."
    )
  }
  sum(w * x) / sum(w)
}

wt_mean <- function(x, w, na.rm = FALSE) {
  if (!is.logical(na.rm) || length(na.rm) != 1) {
    cli::cli_abort("{.arg na.rm} must be a single logical value, not {.obj_type_friendly {na.rm}}.")
  }
  if (length(x) != length(w)) {
    cli::cli_abort(
      "{.arg x} (length {length(x)}) and {.arg w} (length {length(w)}) must be the same length."
    )
  }
  ...
}
```

---

## 9. `stopifnot()` → `cli::cli_abort()` with context

**Rule:** `design-principles.md` §8 — `stopifnot()` gives cryptic errors.

### Original (Ch. 19)
```r
wt_mean <- function(x, w, na.rm = FALSE) {
  stopifnot(is.logical(na.rm), length(na.rm) == 1)
  stopifnot(length(x) == length(w))
  ...
}
```

### Corrected
```r
wt_mean <- function(x, w, na.rm = FALSE) {
  if (!is.logical(na.rm) || length(na.rm) != 1) {
    cli::cli_abort("{.arg na.rm} must be a single {.cls logical} value.")
  }
  if (length(x) != length(w)) {
    cli::cli_abort("{.arg x} and {.arg w} must be the same length.")
  }
  ...
}
```

---

## 10. `ifelse()` → `if_else()`

**Rule:** `anti-patterns.md` §4.2 — `ifelse()` strips attributes and isn't
type-stable.

### Original (Ch. 19)
```r
mtcars %>%
  show_missings() %>%
  mutate(mpg = ifelse(mpg < 20, NA, mpg)) %>%
  show_missings()
```

### Corrected
```r
mtcars |>
  show_missings() |>
  mutate(mpg = if_else(mpg < 20, NA_real_, mpg)) |>
  show_missings()
```

---

## 11. `cat()` in functions → `cli::cli_inform()`

**Rule:** `anti-patterns.md` §1.5 — `cat()` can't be suppressed. Use
`cli::cli_inform()` for messages, or `message()` at minimum.

### Original (Ch. 19)
```r
show_missings <- function(df) {
  n <- sum(is.na(df))
  cat("Missing values: ", n, "\n", sep = "")
  invisible(df)
}

# And the formatting helper:
rule <- function(..., pad = "-") {
  title <- paste0(...)
  width <- getOption("width") - nchar(title) - 5
  cat(title, " ", stringr::str_dup(pad, width), "\n", sep = "")
}
```

### Corrected
```r
show_missings <- function(df) {
  n <- sum(is.na(df))
  cli::cli_inform("Missing values: {.val {n}}")
  invisible(df)
}

rule <- function(..., pad = "-") {
  title <- paste0(...)
  cli::cli_rule(title)
}
```

---

## 12. `size` → `linewidth` in ggplot2

**Rule:** `ggplot2-skill.md` §1 — `size` for line-based geoms was
deprecated in ggplot2 3.4.0.

### Original (Ch. 23, Ch. 25)
```r
ggplot(sim1, aes(x)) +
  geom_point(aes(y = y)) +
  geom_line(aes(y = pred), data = grid, colour = "red", size = 1)

# And:
nz %>%
  add_residuals(nz_mod) %>%
  ggplot(aes(year, resid)) +
  geom_hline(yintercept = 0, colour = "white", size = 3) +
  geom_line()
```

### Corrected
```r
ggplot(sim1, aes(x)) +
  geom_point(aes(y = y)) +
  geom_line(aes(y = pred), data = grid, colour = "red", linewidth = 1)

nz |>
  add_residuals(nz_mod) |>
  ggplot(aes(year, resid)) +
  geom_hline(yintercept = 0, colour = "white", linewidth = 3) +
  geom_line()
```

---

## 13. `data =` and `mapping =` in ggplot2 → positional arguments

**Rule:** `ggplot2-skill.md` — The first two arguments to `ggplot()` are
`data` and `mapping`; naming them is verbose. Similarly for `aes()` with
`x` and `y`.

### Original (Ch. 3 — Data Visualisation)
```r
ggplot(data = mpg) +
  geom_point(mapping = aes(x = displ, y = hwy))

ggplot(data = mpg, mapping = aes(x = displ, y = hwy)) +
  geom_point() +
  geom_smooth()
```

### Corrected
```r
ggplot(mpg, aes(displ, hwy)) +
  geom_point()

ggplot(mpg, aes(displ, hwy)) +
  geom_point() +
  geom_smooth()
```

Note: R4DS 2e already adopts this shorter style.

---

## 14. `1:length(x)` → `seq_along(x)`

**Rule:** `anti-patterns.md` §2.3 — `1:length(x)` produces `1:0` when `x`
is empty, causing bugs. `seq_along()` correctly returns an empty sequence.

### Original (Ch. 21)
```r
# The book shows the problem:
y <- vector("double", 0)
seq_along(y)   # integer(0) — correct
1:length(y)    # 1 0 — wrong!
```

The book correctly identifies this as an anti-pattern and recommends
`seq_along()`. No correction needed — included here because it validates
the positron-verse rule.

---

## 15. `unnest(.drop = TRUE)` → `unnest()` with column selection

**Rule:** `tidyverse-skill.md` §2 — The `.drop` argument was deprecated in
tidyr 1.0.0. Use explicit column selection.

### Original (Ch. 25)
```r
glance <- by_country %>%
  mutate(glance = map(model, broom::glance)) %>%
  unnest(glance, .drop = TRUE)
```

### Corrected
```r
glance <- by_country |>
  mutate(glance = map(model, broom::glance)) |>
  unnest(glance)
```

Columns not selected for unnesting are preserved automatically in modern
tidyr. Use `select()` before or after if you need to drop specific columns.

---

## 16. `summarise_all(list(list))` → `reframe()` or `across()`

**Rule:** `anti-patterns.md` §3 — `summarise_all()` was superseded by
`summarise(across(...))` in dplyr 1.0.0.

### Original (Ch. 25)
```r
mtcars %>%
  group_by(cyl) %>%
  summarise_all(list(list))
```

### Corrected
```r
mtcars |>
  summarise(
    across(everything(), list),
    .by = cyl
  )
```

---

## 17. `facet_wrap(~ var)` → `facet_wrap(vars(var))`

**Rule:** While `~ var` still works, `vars()` is the modern approach that
supports tidy evaluation and multiple variables.

### Original (Ch. 3, Ch. 23)
```r
ggplot(data = mpg) +
  geom_point(mapping = aes(x = displ, y = hwy)) +
  facet_wrap(~ class, nrow = 2)

ggplot(sim5, aes(x, y)) +
  geom_point() +
  geom_line(data = grid, colour = "red") +
  facet_wrap(~ model)
```

### Corrected
```r
ggplot(mpg, aes(displ, hwy)) +
  geom_point() +
  facet_wrap(vars(class), nrow = 2)

ggplot(sim5, aes(x, y)) +
  geom_point() +
  geom_line(data = grid, colour = "red") +
  facet_wrap(vars(model))
```

Note: Both `~ var` and `vars(var)` are valid. The `vars()` form is
preferred when programmatically faceting, but `~ var` is not wrong.

---

## 18. `coord_flip()` → native orientation in ggplot2

**Rule:** Since ggplot2 3.3.0, most geoms support orientation directly,
making `coord_flip()` unnecessary.

### Original (Ch. 3)
```r
ggplot(data = mpg, mapping = aes(x = class, y = hwy)) +
  geom_boxplot() +
  coord_flip()
```

### Corrected
```r
ggplot(mpg, aes(y = class, x = hwy)) +
  geom_boxplot()
```

---

## Summary: What Changed Between R4DS 1e and Current Practice

| Pattern in R4DS 1e | Current Convention | When Changed |
|---|---|---|
| `%>%` (magrittr pipe) | `\|>` (native pipe) | R 4.1.0 (2021) |
| `~.x` formula lambda | `\(x)` anonymous function | R 4.1.0 (2021) |
| `.` pronoun in pipe | Not available in `\|>` | R 4.1.0 (2021) |
| `group_by() \|> summarise()` | `.by` argument | dplyr 1.1.0 (2023) |
| `gather()` / `spread()` | `pivot_longer()` / `pivot_wider()` | tidyr 1.0.0 (2019) |
| `invoke_map()` | `map()` + `exec()` | purrr 1.0.0 (2023) |
| `sapply()` | `map_*()` typed variants | Always (tidyverse) |
| `unnest(.drop =)` | `unnest()` with column selection | tidyr 1.0.0 (2019) |
| `summarise_all()` | `summarise(across())` | dplyr 1.0.0 (2020) |
| `size` in line geoms | `linewidth` | ggplot2 3.4.0 (2022) |
| `coord_flip()` | Native orientation | ggplot2 3.3.0 (2020) |
| `data =`, `mapping =` named | Positional arguments | R4DS 2e convention |
| `stop()` / `stopifnot()` | `cli::cli_abort()` | cli ecosystem (2021+) |
| `cat()` for messages | `cli::cli_inform()` | cli ecosystem (2021+) |
| `ifelse()` | `if_else()` | Always (tidyverse) |

### What R4DS 1e Still Gets Right

The book's core principles remain valid and are embedded in positron-verse:

- Tidy data as the foundation
- `library()` at the top (not `require()` or `pacman`)
- `tibble()` over `data.frame()`
- `seq_along()` over `1:length()`
- Composable pipelines for data transformation
- `stringr` over base R string functions
- Functional programming with `purrr::map()` family
- Explicit `NA` handling with `na.rm = TRUE`
