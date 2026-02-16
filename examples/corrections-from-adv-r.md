# Positron-Verse Corrections — Advanced R (2nd Edition)

> **Source:** https://adv-r.hadley.nz/ (Wickham, 2019)
> **Repo:** https://github.com/hadley/adv-r (main branch, bookdown/Rmd)
> **Scanned:** 2026-02-16 against latest commit (`63ea8d9`)
>
> Advanced R 2e was published in 2019, before R 4.1 (native pipe, lambda
> syntax) and before several tidyverse modernizations. It is a foundational
> text about R internals and programming — many patterns flagged here are
> base R by design, since the book teaches language fundamentals. The
> corrections below focus on patterns where the modern alternative would
> improve the book without changing its pedagogical intent.

---

## 1. `%>%` (magrittr pipe) → `|>` (native pipe)

**Rule:** `CLAUDE.md` invariant #1 — Use `|>` not `%>%`.
**Scope:** 37 occurrences across 9 files.
**Impact:** Eliminates dependency on magrittr; aligns with R 4.1+ standard.

### Original (Functions.Rmd:298-307)
```r
library(magrittr)

x %>%
  deviation() %>%
  square() %>%
  mean() %>%
  sqrt()
```

### Corrected
```r
x |>
  deviation() |>
  square() |>
  mean() |>
  sqrt()
```

### Original (Functionals.Rmd:437-440)
```r
by_cyl %>%
  map(~ lm(mpg ~ wt, data = .x)) %>%
  map(coef) %>%
  map_dbl(2)
```

### Corrected
```r
by_cyl |>
  map(\(df) lm(mpg ~ wt, data = df)) |>
  map(coef) |>
  map_dbl(2)
```

### Original (Introduction.Rmd:225-226)
```r
contributors <- contributors %>%
  filter(login != "hadley") %>%
```

### Corrected
```r
contributors <- contributors |>
  filter(login != "hadley") |>
```

### Files affected
| File | Occurrences |
|------|-------------|
| `Introduction.Rmd` | 8 |
| `Functionals.Rmd` | 7 |
| `Functions.Rmd` | 7 |
| `Big-picture.Rmd` | 4 |
| `Quotation.Rmd` | 4 |
| `Function-operators.Rmd` | 3 |
| `Translation.Rmd` | 2 |
| `Preface.Rmd` | 1 (prose) |
| `OO-tradeoffs.Rmd` | 1 (prose) |

### Prose updates needed
The Functions chapter (§6.3) would need its prose updated to introduce
`|>` as the native pipe rather than `%>%` as the magrittr pipe. The
discussion about `library(magrittr)` and third-party dependency would
change to noting it's now built into R 4.1+.

---

## 2. `~ .x` formula lambdas → `\(x)` anonymous functions

**Rule:** `anti-patterns.md` §3.3 — Use `\(x)` lambda syntax (R 4.1+).
**Scope:** 4 occurrences in Functionals.Rmd.

### Original (Functionals.Rmd:506)
```r
map(df, ~ .x * 2)
modify(df, ~ .x * 2)
df <- modify(df, ~ .x * 2)
```

### Corrected
```r
map(df, \(x) x * 2)
modify(df, \(x) x * 2)
df <- modify(df, \(x) x * 2)
```

### Original (Functionals.Rmd:438)
```r
by_cyl %>%
  map(~ lm(mpg ~ wt, data = .x)) %>%
```

### Corrected
```r
by_cyl |>
  map(\(df) lm(mpg ~ wt, data = df)) |>
```

### Prose update needed
The Functionals chapter introduces `~` formula syntax as purrr shorthand.
This section would need rewriting to teach `\(x)` as the primary lambda
syntax, with a note that `~ .x` is legacy purrr syntax still found in
older code.

---

## 3. `TRUE ~` in `case_when()` → `.default`

**Rule:** `anti-patterns.md` §3.4 — Use `.default` argument (dplyr 1.1.0+).
**Scope:** 2 occurrences.

### Original (Control-flow.Rmd:119-125)
```r
dplyr::case_when(
  x %% 35 == 0 ~ "fizz buzz",
  x %% 5 == 0 ~ "fizz",
  x %% 7 == 0 ~ "buzz",
  is.na(x) ~ "???",
  TRUE ~ as.character(x)
)
```

### Corrected
```r
dplyr::case_when(
  x %% 35 == 0 ~ "fizz buzz",
  x %% 5 == 0 ~ "fizz",
  x %% 7 == 0 ~ "buzz",
  is.na(x) ~ "???",
  .default = as.character(x)
)
```

### Original (Introduction.Rmd:267-270)
```r
y <- case_when(
  x %% 10 == 0 ~ as.character((x %/% 10) %% 10),
  x %% 5 == 0  ~ "+",
  TRUE         ~ "-"
)
```

### Corrected
```r
y <- case_when(
  x %% 10 == 0 ~ as.character((x %/% 10) %% 10),
  x %% 5 == 0  ~ "+",
  .default = "-"
)
```

---

## 4. `ifelse()` → `if_else()` in contributor processing

**Rule:** `anti-patterns.md` §4.2 — Use `dplyr::if_else()` for type safety.

### Original (Introduction.Rmd:229)
```r
desc = ifelse(is.na(name), login, paste0(name, " (", login, ")"))
```

### Corrected
```r
desc = if_else(is.na(name), login, str_glue("{name} ({login})"))
```

Note: The `ifelse()` examples in `Control-flow.Rmd` are teaching base R
vectorization deliberately and should NOT be changed. Only the
contributor-processing code is a correction target.

---

## 5. `ifelse()` → `if_else()` with vctrs caveat in Control-flow.Rmd

**Rule:** This is a prose/recommendation update, not a code change.

### Original (Control-flow.Rmd:114)
```
I recommend using `ifelse()` only when the `yes` and `no` vectors are the
same type as it is otherwise hard to predict the output type. See
<https://vctrs.r-lib.org/articles/stability.html#ifelse> for additional
discussion.
```

### Suggested addition
```
I recommend using `ifelse()` only when the `yes` and `no` vectors are the
same type as it is otherwise hard to predict the output type. See
<https://vctrs.r-lib.org/articles/stability.html#ifelse> for additional
discussion. For tidyverse code, prefer `dplyr::if_else()` which enforces
type consistency and handles missing values explicitly.
```

---

## 6. `paste0()` → `str_c()` / `str_glue()` in contributor processing

**Rule:** `style-conventions.md` §5 — Prefer tidyverse string functions.

### Original (Introduction.Rmd:228-233)
```r
contributors <- contributors %>%
  filter(login != "hadley") %>%
  mutate(
    login = paste0("\\@", login),
    desc = ifelse(is.na(name), login, paste0(name, " (", login, ")"))
  )

cat(paste0(contributors$desc, collapse = ", "))
```

### Corrected
```r
contributors <- contributors |>
  filter(login != "hadley") |>
  mutate(
    login = str_c("\\@", login),
    desc = if_else(is.na(name), login, str_glue("{name} ({login})"))
  )

cat(str_c(contributors$desc, collapse = ", "))
```

Note: `paste0()` used throughout the rest of the book for base R string
operations is intentional — the book teaches R fundamentals, not tidyverse.

---

## 7. `read.csv()` → `readr::read_csv()` in contributor processing

**Rule:** `method-decision-tree.md` §1 — Use readr for data import.

### Original (Introduction.Rmd:224)
```r
contributors <- read.csv("contributors.csv", stringsAsFactors = FALSE)
```

### Corrected
```r
contributors <- readr::read_csv("contributors.csv", show_col_types = FALSE)
```

Note: `stringsAsFactors = FALSE` is no longer needed with readr or with
R 4.0+ (which changed the default). The `read.csv()` uses elsewhere in
the book (Conditions, Perf-improve) are teaching base R intentionally.

---

## 8. `stopifnot()` → `rlang::abort()` with informative messages

**Rule:** `design-principles.md` §8 — The Conditions chapter itself
teaches `rlang::abort()` as superior to `stop()`. But other chapters
use `stopifnot()` extensively.
**Scope:** 23 occurrences across 10 files.

### Original (R6.Rmd:155-156)
```r
initialize = function(name, age = NA) {
  stopifnot(is.character(name), length(name) == 1)
  stopifnot(is.numeric(age), length(age) == 1)
```

### Corrected
```r
initialize = function(name, age = NA) {
  if (!is.character(name) || length(name) != 1) {
    abort("`name` must be a single string.")
  }
  if (!is.numeric(age) || length(age) != 1) {
    abort("`age` must be a single number.")
  }
```

### Original (S3.Rmd:243)
```r
new_percent <- function(x = double()) {
  stopifnot(is.double(x))
```

### Corrected
```r
new_percent <- function(x = double()) {
  if (!is.double(x)) {
    abort("`x` must be a double vector.")
  }
```

### Note
This is a large change that affects the S3, S4, R6, Evaluation, and
Quotation chapters. The Conditions chapter already teaches `abort()` as
the preferred approach. A PR could propose aligning the OOP chapters
with the Conditions chapter's own recommendation.

---

## 9. `stop()` → `rlang::abort()` in non-Conditions chapters

**Rule:** `design-principles.md` §8 — Use `rlang::abort()` / `cli::cli_abort()`.
**Scope:** 20+ occurrences across 7 files (excluding Conditions.Rmd which
already teaches `abort()`).

### Original (Environments.Rmd:367)
```r
where <- function(name, env = caller_env()) {
  if (identical(env, empty_env())) {
    stop("Can't find `", name, "`", call. = FALSE)
  }
```

### Corrected
```r
where <- function(name, env = caller_env()) {
  if (identical(env, empty_env())) {
    abort(glue("Can't find `{name}`."))
  }
```

### Original (R6.Rmd:365)
```r
set = function(name, value) {
  if (name == "random") {
    stop("Can't set `$random`", call. = FALSE)
  }
```

### Corrected
```r
set = function(name, value) {
  if (name == "random") {
    abort("Can't set `$random`.")
  }
```

---

## 10. `data.frame()` → `tibble()` where appropriate

**Rule:** `method-decision-tree.md` §2 — Prefer tibble for data construction.
**Scope:** 75 occurrences across 11 files.
**Caveat:** Most are intentional — the book teaches base R data structures.

### Only the Functionals chapter target is a clear correction:

### Original (Functionals.Rmd:501-504)
```r
df <- data.frame(
  x = 1:3,
  y = 6:4
)
```

### Corrected
```r
df <- tibble(
  x = 1:3,
  y = 6:4
)
```

The Vectors, Subsetting, and Names-values chapters deliberately teach
`data.frame()` internals and should not be changed.

---

## 11. `sapply()` → `map_*()` or `vapply()`

**Rule:** `anti-patterns.md` §2.5 — `sapply()` has unpredictable output type.
**Scope:** 8 occurrences across 3 files.

### Original (Functionals.Rmd — multiple instances)
```r
sapply(mtcars, is.numeric)
```

### Corrected
```r
map_lgl(mtcars, is.numeric)
```

### Note
The Functionals chapter explicitly discusses `sapply()` vs `vapply()` vs
`map_*()`, so these examples are pedagogical. However, the chapter could
emphasize that `map_lgl()` / `map_dbl()` / `map_chr()` are now the
recommended approach.

---

## 12. `library(magrittr)` → remove (native pipe)

**Rule:** `style-conventions.md` §1 — No need for magrittr with native pipe.

### Original (Functions.Rmd:301)
```r
library(magrittr)
```

### Corrected
Remove entirely — `|>` is built into R 4.1+, no package needed.

---

## Summary: Correction Counts by File

| File | `%>%` | `~.x` | `stop()` | `stopifnot()` | `ifelse` | `TRUE~` | `paste0` | Other |
|------|-------|-------|----------|---------------|----------|---------|----------|-------|
| `Introduction.Rmd` | 8 | — | — | — | 1 | 1 | 3 | `read.csv` |
| `Functionals.Rmd` | 7 | 4 | — | — | — | — | — | `sapply`, `data.frame` |
| `Functions.Rmd` | 7 | — | — | — | — | — | — | `library(magrittr)` |
| `Big-picture.Rmd` | 4 | — | — | — | — | — | — | — |
| `Quotation.Rmd` | 4 | — | — | 2 | — | — | — | — |
| `Function-operators.Rmd` | 3 | — | — | — | — | — | — | — |
| `Translation.Rmd` | 2 | — | 1 | — | — | — | — | — |
| `Control-flow.Rmd` | — | — | — | — | (pedagogical) | 1 | — | prose update |
| `R6.Rmd` | — | — | 2 | 3 | — | — | — | — |
| `S3.Rmd` | — | — | — | 6 | — | — | — | — |
| `Environments.Rmd` | — | — | 2 | — | — | — | — | — |
| `Conditions.Rmd` | — | — | (teaches both) | — | — | — | — | already uses `abort()` |
| `Evaluation.Rmd` | — | — | — | 3 | — | — | — | — |

**Total:** ~120 correction instances across 13 files.

---

## What Advanced R Already Gets Right

Despite predating R 4.1, the book has strong foundations:

| Convention | Status |
|-----------|--------|
| `<-` for assignment | Consistent throughout |
| `TRUE`/`FALSE` (never `T`/`F`) | Clean |
| `library()` at top of chapters | Clean |
| `rlang::abort()` taught | Yes — Conditions chapter |
| `tibble()` for tidy data | Used where appropriate |
| No `attach()` | Clean |
| `snake_case` naming | Consistent |
| Functional programming emphasis | Core theme |
| `vapply()` over `sapply()` taught | Yes — but examples still use `sapply()` |
| Custom conditions with metadata | Extensively covered |

---

## Comparison: Advanced R vs R4DS Modernization Status

| Pattern | Adv-R 2e (2019) | R4DS 1e (2017) | R4DS 2e (2023) |
|---------|----------------|----------------|----------------|
| `%>%` pipe | 37 uses | Everywhere | 0 — all `\|>` |
| `~.x` lambdas | 4 uses | Everywhere | 0 — all `\(x)` |
| `stop()` | ~20 uses | Some | 0 |
| `stopifnot()` | 23 uses | Some | 0 |
| `ifelse()` | 1 + pedagogical | Used | 0 (1 stray in intro) |
| `TRUE ~` in case_when | 2 | Not used | 0 |
| `paste0()` | 63 uses | Common | 1 stray |
| `sapply()` | 8 uses | Used | 0 (only base-R chapter) |
| `data.frame()` | 75 uses | — | 0 (only base-R chapter) |
| `read.csv()` | 1 + pedagogical | — | 0 |
| `library(magrittr)` | 1 | Implied | 0 |

**Bottom line:** Advanced R 2e has ~120 correction targets vs R4DS 2e's 4.
This reflects its age (2019 vs 2023) and its focus on base R internals.
Many patterns are deliberately teaching base R, so corrections should be
selective — prioritize the introduction/contributor code, the Functionals
chapter, and cross-referencing the Conditions chapter's own `abort()`
recommendation in other chapters.
