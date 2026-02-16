# Positron-Verse Corrections — R Packages (2nd Edition)

> **Source:** https://r-pkgs.org/ (Wickham & Bryan, 2023)
> **Repo:** https://github.com/hadley/r-pkgs (main branch, bookdown/Rmd)
> **Scanned:** 2026-02-16 against latest commit (`3e497c8`)
>
> R Packages 2e was substantially rewritten in 2023 and is largely modern.
> It already uses `|>` in most places, `\(x)` lambdas, `cli` for user
> messaging guidance, and teaches `rlang::abort()` as best practice. The
> corrections below are residual patterns — mostly in quoted external
> examples, pedagogical "before" code, and a few spots where the book's
> own code doesn't follow its own advice.

---

## 1. `%>%` (magrittr pipe) → `|>` (native pipe)

**Rule:** `CLAUDE.md` invariant #1 — Use `|>` not `%>%`.
**Scope:** ~17 code occurrences across 4 files (plus 1 prose mention).

### Correctable code

#### license.Rmd (8 pipes, lines 42–62)

```r
# Original
parsed <- packages %>%
  select(package = Package, license = License) %>%
  mutate(...)

parsed %>% count(license, sort = TRUE)

copyleft <- parsed %>%
  filter(str_detect(license, "GPL")) %>%
  filter(!str_detect(license, "LGPL")) %>%
  count(license, sort = TRUE)

permissive <- parsed %>%
  count(license, sort = TRUE) %>%
  anti_join(copyleft) %>%
  filter(...)
```

```r
# Corrected
parsed <- packages |>
  select(package = Package, license = License) |>
  mutate(...)

parsed |> count(license, sort = TRUE)

copyleft <- parsed |>
  filter(str_detect(license, "GPL")) |>
  filter(!str_detect(license, "LGPL")) |>
  count(license, sort = TRUE)

permissive <- parsed |>
  count(license, sort = TRUE) |>
  anti_join(copyleft) |>
  filter(...)
```

#### package-within.Rmd (4 pipes, lines 227–281)

```r
# Original
dat <- dat %>%
  localize_beach() %>%
  celsify_temp()
```

```r
# Corrected
dat <- dat |>
  localize_beach() |>
  celsify_temp()
```

This pattern appears twice (lines 227–229 and 280–282). Both are the book's own example code (not quoted from external packages).

### Quoted from external packages (not correctable)

| File | Lines | Source |
|------|-------|--------|
| `man.Rmd` | 677–684 | Quoted tidyr roxygen `@examples` (chop/unchop) |
| `man.Rmd` | 783 | Quoted dbplyr roxygen `@examples` |
| `man.Rmd` | 842 | Quoted googledrive roxygen `@examples` |

These reproduce verbatim documentation from other packages to illustrate
roxygen conventions. They should match the source package, not be
modernized independently.

### Prose mention (informational, not correctable)

`dependencies-in-practice.Rmd:137` mentions `%>%` from magrittr as an
example of re-exporting. This is accurate description, not a code pattern.

---

## 2. `stopifnot()` → informative error with `cli::cli_abort()`

**Rule:** `design-principles.md` §8 — Prefer informative error messages.
**Scope:** 2 occurrences in whole-game.Rmd (the book's flagship tutorial function).

### Original (whole-game.Rmd:891–892, repeated at 942–943)

```r
str_split_one <- function(string, pattern, n = Inf) {
  stopifnot(is.character(string), length(string) <= 1)
  if (length(string) == 1) {
    stringr::str_split(string = string, pattern = pattern, n = n)[[1]]
  } else {
    character()
  }
}
```

### Corrected

```r
str_split_one <- function(string, pattern, n = Inf) {
  if (!is.character(string) || length(string) > 1) {
    cli::cli_abort("{.arg string} must be a single string or empty.")
  }
  if (length(string) == 1) {
    stringr::str_split(string = string, pattern = pattern, n = n)[[1]]
  } else {
    character()
  }
}
```

### Note

This is the tutorial's primary example function — `str_split_one()` — which
appears in the "whole game" chapter that walks through creating a complete
package. The book's own Conditions guidance (and `code.Rmd` §9 on
user-facing messages) recommends `cli::cli_abort()`. Aligning the flagship
example with the book's own recommendation would strengthen the pedagogy.

---

## 3. `stop()` → `rlang::abort()` / `cli::cli_abort()`

**Rule:** `design-principles.md` §8 — Use structured error signaling.
**Scope:** 2 occurrences.

### Original (dependencies-in-practice.Rmd:307–312)

```r
my_fun <- function(a, b) {
  if (!requireNamespace("aaapkg", quietly = TRUE)) {
    stop(
      "Package \"aaapkg\" must be installed to use this function.",
      call. = FALSE
    )
  }
  # code that includes calls such as aaapkg::aaa_fun()
}
```

### Corrected

```r
my_fun <- function(a, b) {
  rlang::check_installed("aaapkg", reason = "to use this function.")
  # code that includes calls such as aaapkg::aaa_fun()
}
```

### Note

The book itself recommends `rlang::check_installed()` later in the same
chapter (and references it at lifecycle.Rmd:650). The `stop()` version
appears first as the "manual" approach but could be updated to show the
recommended pattern as primary.

### Original (lifecycle.Rmd:644)

```r
your_new_function <- function(...) {
  if (packageVersion("otherpkg") < "1.0.0") {
    stop("otherpkg >= 1.0.0 needed for this function.")
  }
  # the rest of the function
}
```

### Corrected

```r
your_new_function <- function(...) {
  rlang::check_installed("otherpkg", version = "1.0.0")
  # the rest of the function
}
```

---

## 4. `message()` → `cli::cli_inform()`

**Rule:** `design-principles.md` §8 — Use cli for user-facing messages.
**Scope:** 2 code occurrences.

### Original (lifecycle.Rmd:636)

```r
your_existing_function <- function(..., cool_new_feature = FALSE) {
  if (isTRUE(cool_new_feature) && packageVersion("otherpkg") < "1.0.0") {
    message("otherpkg >= 1.0.0 is needed for cool_new_feature")
    cool_new_feature <- FALSE
  }
  # the rest of the function
}
```

### Corrected

```r
your_existing_function <- function(..., cool_new_feature = FALSE) {
  if (isTRUE(cool_new_feature) && packageVersion("otherpkg") < "1.0.0") {
    cli::cli_inform("{.pkg otherpkg} >= 1.0.0 is needed for {.arg cool_new_feature}.")
    cool_new_feature <- FALSE
  }
  # the rest of the function
}
```

### Original (vignettes.Rmd:328)

```r
message("No token available. Code chunks will not be evaluated.")
```

### Corrected

```r
cli::cli_inform("No token available. Code chunks will not be evaluated.")
```

### Note

The book's own `code.Rmd` chapter extensively teaches cli as the preferred
approach for user-facing messages. These examples could be aligned.

---

## 5. `paste0()` → `glue::glue()` in generated file paths

**Rule:** `style-conventions.md` §5 — Prefer glue for string interpolation.
**Scope:** ~5 occurrences in package-within.Rmd.

### Original (package-within.Rmd:94)

```r
(outfile <- paste0(timestamp, "_", sub("(.*)([.]csv$)", "\\1_clean\\2", infile)))
```

### Corrected

```r
(outfile <- glue::glue("{timestamp}_{sub('(.*)([.]csv$)', '\\\\1_clean\\\\2', infile)}"))
```

### Note — Intentionally NOT corrected

The `paste0()` uses in package-within.Rmd are part of a progressive
refactoring narrative. The chapter deliberately shows a messy script (using
`read.csv`, `paste0`, base R) and evolves it into a proper package. The
"before" code uses base R patterns intentionally — the "after" code at
lines 442+ introduces cleaner alternatives. Correcting the "before" code
would undermine the pedagogical arc.

The `paste0()` in `whole-game.Rmd:272` is book infrastructure for displaying
git output and is not user-facing code.

---

## 6. `read.csv()` → `readr::read_csv()` (selective)

**Rule:** `method-decision-tree.md` §1 — Use readr for data import.
**Scope:** 3 code occurrences across 2 files.

### Correctable

#### vignettes.Rmd:250

```r
# Original (quoted from tidyr vignette)
weather <- as_tibble(read.csv("weather.csv", stringsAsFactors = FALSE))
```

```r
# Corrected
weather <- read_csv("weather.csv", show_col_types = FALSE)
```

Note: `stringsAsFactors = FALSE` has been the default since R 4.0 (2020),
and `readr::read_csv()` never had this issue. The `as_tibble()` wrapper is
also unnecessary with readr.

### Intentionally NOT corrected

`package-within.Rmd:53,58` — The `read.csv()` calls are the "messy
script" being refactored into a package. The narrative evolves this to
`readr::read_csv()` at line 225. Changing the "before" code would break
the teaching progression.

---

## 7. `sapply()` → `map_int()` or `vapply()`

**Rule:** `anti-patterns.md` §2.5 — `sapply()` has unpredictable output type.
**Scope:** 1 occurrence.

### Original (dependencies-mindset-background.Rmd:173)

```r
n_hard_deps <- function(pkg) {
  deps <- tools::package_dependencies(pkg, recursive = TRUE)
  sapply(deps, length)
}
```

### Corrected

```r
n_hard_deps <- function(pkg) {
  deps <- tools::package_dependencies(pkg, recursive = TRUE)
  purrr::map_int(deps, length)
}
```

### Note

This is a utility function the book uses to count dependencies. Since it
appears in a chapter about dependency management, `vapply(deps, length, 0L)`
would also be acceptable (avoiding an extra dependency). However, since the
book already depends on purrr, `map_int()` is cleaner.

---

## 8. `data.frame()` → `tibble()` (selective)

**Rule:** `method-decision-tree.md` §2 — Prefer tibble for data construction.
**Scope:** 5 code occurrences in testing-design.Rmd; 1 in man.Rmd.

### testing-design.Rmd (lines 113, 121, 137, 146–147)

These create simple test fixtures inside `test_that()` blocks:

```r
dat <- data.frame(x = c("a", "b", "c"), y = c(1, 2, 3))
dat2 <- data.frame(x = c("x", "y", "z"), y = c(4, 5, 6))
```

### Note — Intentionally NOT corrected

Test fixture code in packages should minimize dependencies. Using
`data.frame()` in test code is a defensible choice — it avoids requiring
tibble as a test dependency. The book is teaching testing patterns, not
data construction. These should remain as-is.

### man.Rmd:780 (quoted dbplyr example)

```r
#' df <- data.frame(x = 1, y = 2)
```

This is quoted from dbplyr's roxygen documentation and should match the
source package.

---

## Summary: Correction Counts by File

| File | `%>%` | `stop()` | `stopifnot()` | `message()` | `paste0` | `read.csv` | `sapply` | Other |
|------|-------|----------|---------------|-------------|----------|------------|---------|-------|
| `license.Rmd` | 8 | — | — | — | — | — | — | — |
| `package-within.Rmd` | 4 | — | — | — | (pedagogical) | (pedagogical) | — | — |
| `whole-game.Rmd` | — | — | 2 | — | — | — | — | — |
| `dependencies-in-practice.Rmd` | (prose) | 1 | — | — | — | — | — | — |
| `lifecycle.Rmd` | — | 1 | — | 1 | — | — | — | — |
| `vignettes.Rmd` | — | — | — | 1 | — | 1 | — | — |
| `dependencies-mindset-background.Rmd` | — | — | — | — | — | — | 1 | — |
| `man.Rmd` | (quoted) | — | — | — | — | — | — | — |
| `testing-design.Rmd` | — | — | — | — | — | — | — | `data.frame` (intentional) |

**Total:** ~20 actionable correction instances across 7 files.

---

## What R Packages 2e Already Gets Right

The 2023 rewrite is exemplary — it practices what it preaches:

| Convention | Status |
|-----------|--------|
| `|>` native pipe | Used throughout (residual `%>%` only in 2 files + quoted examples) |
| `\(x)` lambda syntax | Consistent — no formula lambdas found |
| `<-` for assignment | Clean throughout |
| `TRUE`/`FALSE` (never `T`/`F`) | Clean (R-CMD-check.Rmd even warns against `T`/`F`) |
| `cli` for user messages | Extensively taught in code.Rmd |
| `rlang::abort()` / `cli::cli_abort()` | Taught as best practice |
| `rlang::check_installed()` | Recommended for suggested-package checks |
| `glue()` for string interpolation | Used where appropriate |
| No `gather()`/`spread()` | Only mentioned as superseded examples |
| No `map_dfr()`/`map_dfc()` | Clean |
| No `.default` issue in `case_when()` | No `case_when()` used |
| `snake_case` naming | Consistent |
| Package-centric `::` qualification | Extensively taught |
| `roxygen2` documentation | Core teaching tool |
| `testthat` 3e with `expect_snapshot()` | Fully modern |
| `usethis` workflow | Standard throughout |

---

## Comparison: R Packages 2e vs Other Books

| Pattern | R Pkgs 2e (2023) | R4DS 2e (2023) | Adv-R 2e (2019) |
|---------|-----------------|----------------|-----------------|
| `%>%` pipe | ~17 residual | 0 (1 stray) | 37 |
| `~.x` lambdas | 0 | 0 | 4 |
| `stop()` | 2 | 0 | 20+ |
| `stopifnot()` | 2 | 0 | 23 |
| `message()` | 2 | 0 | — |
| `paste0()` | ~5 (pedagogical) | 1 | 63 |
| `sapply()` | 1 | 0 | 8 |
| `read.csv()` | 1 + pedagogical | 0 | 1 + pedagogical |
| `data.frame()` | 5 (test fixtures) | 0 | 75 |
| Total actionable | **~20** | **4** | **~120** |

**Bottom line:** R Packages 2e is in excellent shape — comparable to R4DS 2e.
The ~20 actionable items are mostly residual pipes in `license.Rmd`, the
flagship `str_split_one()` function using `stopifnot()`, and a few spots
where `stop()`/`message()` could use the cli/rlang patterns the book itself
teaches. Many other patterns (paste0, read.csv, data.frame) are intentionally
part of "before/after" pedagogical narratives and should not be changed.
