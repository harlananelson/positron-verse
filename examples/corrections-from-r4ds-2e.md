# Positron-Verse Corrections — R for Data Science (2nd Edition)

> **Source:** https://r4ds.hadley.nz/ (Wickham, Çetinkaya-Rundel, Grolemund, 2023)
> **Repo:** https://github.com/hadley/r4ds (main branch)
> **Scanned:** 2026-02-16 against bleeding-edge main (commit `aea4641`)
>
> R4DS 2e is a major rewrite that already incorporates most modern conventions:
> native pipe `|>`, lambda `\(x)`, `pivot_longer()`/`pivot_wider()`,
> `linewidth`, `if_else()`, and no deprecated purrr functions. Only 4
> corrections were found across 39 chapter files (~19,000 lines).

---

## 1. `ifelse()` → `if_else()` in intro contributor processing

**Rule:** `anti-patterns.md` §4.2 — The book itself teaches this in Ch. 12
(Logicals): "dplyr's `if_else()` is very similar to base R's `ifelse()`.
There are two main advantages..." Yet the intro chapter's own code uses
the discouraged form.

**File:** `intro.qmd` line 307

### Original
```r
contributors <- contributors |>
  filter(!login %in% c("hadley", "garrettgman", "mine-cetinkaya-rundel")) |>
  mutate(
    login = paste0("\\@", login),
    desc = ifelse(is.na(name), login, paste0(name, " (", login, ")"))
  )
```

### Corrected
```r
contributors <- contributors |>
  filter(!login %in% c("hadley", "garrettgman", "mine-cetinkaya-rundel")) |>
  mutate(
    login = str_c("\\@", login),
    desc = if_else(is.na(name), login, str_glue("{name} ({login})"))
  )
```

---

## 2. `paste0()` → `str_c()` / `str_glue()` in intro contributor processing

**Rule:** `style-conventions.md` §5 — Ch. 14 (Strings) teaches `str_c()` as
the tidyverse replacement for `paste0()` and `str_glue()` for interpolation.
The intro chapter uses the base R forms in the same code block.

**File:** `intro.qmd` lines 306, 311

### Original
```r
login = paste0("\\@", login),
...
cat(paste0(contributors$desc, collapse = ", "))
```

### Corrected
```r
login = str_c("\\@", login),
...
cat(str_c(contributors$desc, collapse = ", "))
```

---

## 3. Named `data=` / `mapping=` in coord example (inconsistency)

**Rule:** `ggplot2-skill.md` — Use positional arguments for `ggplot()` and
`aes()`. The rest of the layers chapter already does this; this one example
was missed.

**File:** `layers.qmd` lines 927-929

### Original
```r
bar <- ggplot(data = diamonds) +
  geom_bar(
    mapping = aes(x = clarity, fill = clarity),
    show.legend = FALSE,
    width = 1
  ) +
  theme(aspect.ratio = 1)
```

### Corrected
```r
bar <- ggplot(diamonds) +
  geom_bar(
    aes(x = clarity, fill = clarity),
    show.legend = FALSE,
    width = 1
  ) +
  theme(aspect.ratio = 1)
```

---

## 4. `summarise()` → `summarize()` (spelling consistency)

**Rule:** Internal consistency — The book uses `summarize()` (American
spelling) in 113 occurrences and notes in a Ch. 3 footnote that
`summarise()` is the British alternative. One instance in the arrow
chapter uses the British spelling.

**File:** `arrow.qmd` line 112

### Original
```r
seattle_csv |>
  group_by(CheckoutYear) |>
  summarise(Checkouts = sum(Checkouts)) |>
  arrange(CheckoutYear) |>
  collect()
```

### Corrected
```r
seattle_csv |>
  group_by(CheckoutYear) |>
  summarize(Checkouts = sum(Checkouts)) |>
  arrange(CheckoutYear) |>
  collect()
```

---

## What R4DS 2e Already Gets Right

The 2nd edition is a model of modern R style. A comprehensive scan of all
39 chapter files confirmed:

| Convention | Status | Notes |
|-----------|--------|-------|
| `\|>` native pipe | All code | No `%>%` in any executable code |
| `\(x)` lambda syntax | All code | `~.x` mentioned only in a footnote as legacy |
| `pivot_longer()`/`pivot_wider()` | All code | No `gather()`/`spread()` |
| `linewidth` for line geoms | All code | `size` used correctly only for points |
| `if_else()` over `ifelse()` | All code except intro.qmd | Taught explicitly in Ch. 12 |
| `separate_wider_delim()` | Used where appropriate | Superseded `separate()` |
| No `map_dfr()`/`map_dfc()` | Clean | No deprecated purrr functions |
| No `invoke_map()` | Clean | Removed entirely |
| `color` not `colour` | Consistent | American spelling throughout |
| No `T`/`F` | Clean | Always `TRUE`/`FALSE` |
| No `=` for assignment | Clean | Always `<-` |
| `tibble()` not `data.frame()` | Appropriate | `data.frame()` only in base-R chapter (pedagogical) |
| `library()` not `require()` | Clean | No `require()` calls |
| No `sapply()` in tidyverse code | Clean | `sapply()` only in base-R chapter (pedagogical) |

---

## Patterns Present But Intentionally Not Corrected

These patterns exist in the book but are **pedagogically intentional**:

### `group_by()` instead of `.by`
- 113+ uses of `group_by() |> summarize()`
- `.by` (dplyr 1.1.0, Jan 2023) is never mentioned
- **Why not corrected:** The book teaches `group_by()` as a fundamental
  concept. Converting to `.by` would require restructuring how grouping
  is taught. Better submitted as an issue proposing a new section than
  as a code change.

### `coord_flip()` in layers.qmd and EDA.qmd exercise
- Used to demonstrate coordinate system transformations
- **Why not corrected:** The pedagogical point is about coordinate
  transforms as a concept, not about making horizontal plots. The
  exercise explicitly asks students to compare `coord_flip()` with
  axis swapping.

### `data =` / `mapping =` named args in data-visualize.qmd
- Used in the very first ggplot2 introduction
- **Why not corrected:** The chapter deliberately introduces named
  arguments first, then transitions to positional arguments later.
  This scaffolding is intentional pedagogy.

### `facet_wrap(~var)` formula syntax
- Used throughout instead of `facet_wrap(vars(var))`
- **Why not corrected:** Both forms are valid. The formula syntax
  is more concise and widely used. `vars()` is preferred only for
  programmatic faceting.

### `.groups = "drop"` in summarize calls
- 16 occurrences across multiple chapters
- **Why not corrected:** Would require teaching `.by` first, which
  restructures the grouping narrative. Better as a holistic proposal.

### `cat()` for Quarto `results: asis` output
- Used in intro.qmd and quarto.qmd
- **Why not corrected:** `cat()` is the correct tool for emitting raw
  markdown in Quarto chunks with `results: asis`.

---

## Comparison: R4DS 1e vs 2e Modernization

| Pattern | 1e Status | 2e Status |
|---------|-----------|-----------|
| `%>%` pipe | Everywhere | Gone — all `\|>` |
| `~.x` lambdas | Everywhere | Gone — all `\(x)` |
| `gather()`/`spread()` | Primary tools | Gone — `pivot_*()` |
| `ifelse()` | Used in code | 1 stray instance |
| `invoke_map()` | Used in code | Gone |
| `map_dfr()` | Used in code | Gone |
| `sapply()` | Used freely | Only in base-R chapter |
| `size` for lines | Used in code | Gone — all `linewidth` |
| `data=`/`mapping=` verbose | Default style | 1 stray instance |
| `.by` grouping | N/A (didn't exist) | Not yet adopted |
| `stop()` for errors | Used in code | Not present (no error-handling section) |
| `paste0()` in code | Common | 1 stray instance |

**Bottom line:** R4DS 2e addressed nearly every 1e anti-pattern. The 4
corrections found are edge cases, not systemic issues.
