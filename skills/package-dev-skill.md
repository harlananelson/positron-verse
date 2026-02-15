# R Package Development Skill

> **Usage:** Load this file into Claude Code (via CLAUDE.md pointer) or paste into an LLM session for R package development following Posit conventions.

Comprehensive reference for R package development using devtools, usethis, testthat, and roxygen2. Based on Hadley Wickham and Jenny Bryan's *R Packages (2e)* (https://r-pkgs.org/).

---

## 1) Creating a Package

```r
usethis::create_package("~/projects/mypackage")

# Package structure created:
# mypackage/
# ├── DESCRIPTION
# ├── NAMESPACE
# ├── R/
# └── mypackage.Rproj
```

### Essential Setup Steps
```r
usethis::use_git()                    # Initialize git
usethis::use_github()                 # Push to GitHub
usethis::use_mit_license()            # Or use_gpl3_license(), etc.
usethis::use_readme_rmd()             # README with code examples
usethis::use_testthat()               # Testing infrastructure
usethis::use_pipe()                   # Re-export |> (or %>%)
usethis::use_package_doc()            # Package-level documentation
usethis::use_news_md()                # Changelog
usethis::use_pkgdown()                # Package website
```

---

## 2) DESCRIPTION File

```
Package: mypackage
Title: What the Package Does (One Line, Title Case)
Version: 0.1.0
Authors@R:
    person("First", "Last", email = "email@example.com",
           role = c("aut", "cre"),
           comment = c(ORCID = "0000-0000-0000-0000"))
Description: A paragraph describing the package. Indent continuation lines
    with 4 spaces. Use full sentences.
License: MIT + file LICENSE
Encoding: UTF-8
Roxygen: list(markdown = TRUE)
RoxygenNote: 7.3.1
Imports:
    dplyr (>= 1.1.0),
    rlang (>= 1.0.0)
Suggests:
    testthat (>= 3.0.0),
    knitr,
    rmarkdown
Config/testthat/edition: 3
URL: https://github.com/user/mypackage
BugReports: https://github.com/user/mypackage/issues
```

### Adding Dependencies
```r
usethis::use_package("dplyr")          # Adds to Imports
usethis::use_package("ggplot2", "Suggests")  # Adds to Suggests
usethis::use_dev_package("newpkg", remote = "user/newpkg")  # GitHub-only
```

### Imports vs Suggests

| Imports | Suggests |
|---------|----------|
| Required to install your package | Optional |
| Functions you use directly | Testing, vignettes, examples |
| Declared in NAMESPACE | Wrapped in `if (requireNamespace())` |

---

## 3) Writing Functions

### File Organization
- One function per file (or a family of closely related functions)
- File name matches primary function: `compute_metrics.R`
- Helper functions can share a file with their parent

### roxygen2 Documentation
```r
#' Compute summary metrics for a dataset
#'
#' Calculates mean, median, and standard deviation for numeric columns,
#' optionally grouped by a categorical variable.
#'
#' @param data A data frame or tibble.
#' @param group_var <[`tidy-select`][dplyr::dplyr_tidy_select]> Optional
#'   grouping variable.
#' @param na.rm Logical. Should missing values be removed? Defaults to `TRUE`.
#'
#' @returns A tibble with summary statistics. Contains columns:
#'   - `variable`: Column name
#'   - `mean`, `median`, `sd`: Summary statistics
#'
#' @export
#'
#' @examples
#' compute_metrics(mtcars)
#' compute_metrics(mtcars, group_var = cyl)
compute_metrics <- function(data, group_var = NULL, na.rm = TRUE) {
  check_data_frame(data)

  data |>
    dplyr::group_by({{ group_var }}) |>
    dplyr::summarize(
      dplyr::across(
        where(is.numeric),
        list(mean = \(x) mean(x, na.rm = na.rm),
             sd = \(x) sd(x, na.rm = na.rm))
      )
    )
}
```

### Key roxygen2 Tags

| Tag | Purpose |
|-----|---------|
| `@param name Description` | Document parameter |
| `@returns Description` | Describe return value |
| `@export` | Make available to users |
| `@examples` | Runnable examples |
| `@importFrom pkg fun` | Import specific function |
| `@seealso` | Related functions |
| `@family topic` | Group related functions |
| `@inheritParams other_fun` | Copy param docs |
| `@noRd` | Internal function (no .Rd file) |

### Referencing Other Packages
```r
# In function body — use namespace qualification
dplyr::filter(data, condition)
rlang::abort("Error message")

# Or import specific functions (in roxygen2 header)
#' @importFrom dplyr filter mutate
#' @importFrom rlang abort
```

**Convention:** Use `pkg::fun()` for 1-2 uses; `@importFrom` for frequently used functions.

---

## 4) Testing (testthat 3e)

### Setup
```r
usethis::use_testthat(edition = 3)
usethis::use_test("compute_metrics")  # Creates tests/testthat/test-compute_metrics.R
```

### Writing Tests
```r
test_that("compute_metrics returns correct structure", {
  result <- compute_metrics(mtcars)

  expect_s3_class(result, "tbl_df")
  expect_true("mpg_mean" %in% names(result))
  expect_true("mpg_sd" %in% names(result))
})

test_that("compute_metrics handles grouping", {
  result <- compute_metrics(mtcars, group_var = cyl)

  expect_equal(nrow(result), 3)  # 3 unique cyl values
})

test_that("compute_metrics errors on non-data-frame", {
  expect_error(compute_metrics("not a df"))
})

test_that("compute_metrics handles NAs", {
  df <- tibble::tibble(x = c(1, 2, NA))
  result <- compute_metrics(df)

  expect_false(is.na(result$x_mean))
})
```

### Snapshot Tests
```r
test_that("error messages are informative", {
  expect_snapshot(error = TRUE, {
    compute_metrics("not a df")
  })
})

test_that("output format is stable", {
  expect_snapshot({
    compute_metrics(mtcars[1:5, 1:3])
  })
})
```

### Running Tests
```r
devtools::test()             # Run all tests
devtools::test_active_file() # Run tests for current file
testthat::test_file("tests/testthat/test-compute_metrics.R")
```

### Test Expectations

| Function | Tests |
|----------|-------|
| `expect_equal(x, y)` | Values equal (with tolerance) |
| `expect_identical(x, y)` | Exact match |
| `expect_true(x)` | Logical TRUE |
| `expect_false(x)` | Logical FALSE |
| `expect_null(x)` | NULL |
| `expect_length(x, n)` | Length |
| `expect_named(x, names)` | Names |
| `expect_s3_class(x, "class")` | S3 class |
| `expect_error(expr, "msg")` | Error raised |
| `expect_warning(expr)` | Warning raised |
| `expect_message(expr)` | Message raised |
| `expect_snapshot(expr)` | Output matches snapshot |
| `expect_no_error(expr)` | No error raised |

---

## 5) Error Handling (rlang + cli)

```r
#' @importFrom rlang abort caller_env caller_arg
#' @importFrom cli cli_abort cli_warn cli_inform

# Input validation with informative errors
check_data_frame <- function(x,
                             arg = caller_arg(x),
                             call = caller_env()) {
  if (!is.data.frame(x)) {
    cli_abort(
      "{.arg {arg}} must be a data frame, not {.obj_type_friendly {x}}.",
      call = call
    )
  }
}

check_string <- function(x,
                         arg = caller_arg(x),
                         call = caller_env()) {
  if (!rlang::is_string(x)) {
    cli_abort(
      "{.arg {arg}} must be a single string.",
      call = call
    )
  }
}
```

### cli Formatting
```r
cli_abort("Column {.field {col_name}} not found in {.arg data}.")
cli_warn("NAs introduced in {.fn compute_metrics}.")
cli_inform("Processing {.val {n}} observations.")

# Inline markup
# {.arg x}     → argument name
# {.fn mean}   → function name
# {.field col}  → column/field name
# {.val 42}    → value
# {.file path} → file path
# {.code expr} → code expression
# {.obj_type_friendly x} → friendly type name
```

---

## 6) Development Workflow

### The devtools Loop
```r
# 1. Edit R/ files
# 2. Load for testing
devtools::load_all()    # Ctrl+Shift+L in RStudio

# 3. Run tests
devtools::test()        # Ctrl+Shift+T

# 4. Rebuild documentation
devtools::document()    # Ctrl+Shift+D

# 5. Check package
devtools::check()       # Ctrl+Shift+E
```

### Checking
```r
devtools::check()    # Full R CMD check (run before every commit)

# Zero errors, zero warnings, zero notes = CRAN ready
```

---

## 7) Vignettes

```r
usethis::use_vignette("getting-started")

# Creates vignettes/getting-started.Rmd with header:
# ---
# title: "Getting Started"
# output: rmarkdown::html_vignette
# vignette: >
#   %\VignetteIndexEntry{Getting Started}
#   %\VignetteEngine{knitr::rmarkdown}
#   %\VignetteEncoding{UTF-8}
# ---
```

---

## 8) CI/CD

```r
# GitHub Actions
usethis::use_github_action_check_standard()  # R CMD check on multiple OS
usethis::use_github_action("test-coverage")  # Code coverage
usethis::use_github_action("pkgdown")        # Auto-build website

# Code coverage
usethis::use_coverage(type = "codecov")
```

---

## 9) Package Website (pkgdown)

```r
usethis::use_pkgdown()
pkgdown::build_site()

# Customize in _pkgdown.yml:
# url: https://user.github.io/mypackage
# template:
#   bootstrap: 5
# reference:
#   - title: Main functions
#     contents:
#       - compute_metrics
#       - plot_results
```

---

## 10) Key References

- **R Packages (2e)**: https://r-pkgs.org/
- **testthat**: https://testthat.r-lib.org/
- **roxygen2**: https://roxygen2.r-lib.org/
- **usethis**: https://usethis.r-lib.org/
- **devtools**: https://devtools.r-lib.org/
- **pkgdown**: https://pkgdown.r-lib.org/
- **cli**: https://cli.r-lib.org/
