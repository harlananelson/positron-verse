# R4DS 2e — Potential PR Candidates

> **Repo:** https://github.com/hadley/r4ds
> **Branch reviewed:** `main` (as of 2026-02-15)
> **Contribution guide:** See `contribute.qmd` — typo-level fixes welcome,
> significant changes require copyright assignment phrase.

Ranked by likelihood of acceptance (most likely first).

---

## 1. `summarise()` → `summarize()` consistency in arrow.qmd

**File:** `arrow.qmd` line 112
**Risk:** Very low — pure consistency fix
**Why it's good:** The book uses `summarize()` (American spelling) in all
113 other occurrences and explicitly notes in Ch. 3 footnote that
`summarise()` is the British alternative. One instance slipped through.

```r
# Before
seattle_csv |>
  group_by(CheckoutYear) |>
  summarise(Checkouts = sum(Checkouts)) |>

# After
seattle_csv |>
  group_by(CheckoutYear) |>
  summarize(Checkouts = sum(Checkouts)) |>
```

---

## 2. `ifelse()` → `if_else()` in intro.qmd

**File:** `intro.qmd` line 307
**Risk:** Very low — the book itself argues for this change
**Why it's good:** The logicals chapter (Ch. 12) explicitly teaches that
`if_else()` is preferred over `ifelse()` for type safety. The intro
chapter's own contributor-processing code uses the discouraged form.

```r
# Before
desc = ifelse(is.na(name), login, paste0(name, " (", login, ")"))

# After
desc = if_else(is.na(name), login, paste0(name, " (", login, ")"))
```

---

## 3. `paste0()` → `str_c()` / `str_glue()` in intro.qmd

**File:** `intro.qmd` lines 306-311
**Risk:** Low — the strings chapter teaches this preference
**Why it's good:** Ch. 14 (Strings) introduces `str_c()` as the tidyverse
replacement for `paste0()` and `str_glue()` for interpolation. The intro
uses the base R forms. Could be combined with #2 as a single PR.

```r
# Before
login = paste0("\\@", login),
desc = ifelse(is.na(name), login, paste0(name, " (", login, ")"))

# After
login = str_c("\\@", login),
desc = if_else(is.na(name), login, str_glue("{name} ({login})"))
```

---

## 4. Named `data=`/`mapping=` args in layers.qmd coord example

**File:** `layers.qmd` lines 927-929
**Risk:** Low — inconsistent with the rest of the chapter
**Why it's good:** Every other ggplot call in the chapter uses positional
args (`ggplot(diamonds, aes(...))`). This one coord_polar/coord_flip
example still uses the verbose named form from the 1e style.

```r
# Before
bar <- ggplot(data = diamonds) +
  geom_bar(
    mapping = aes(x = clarity, fill = clarity),

# After
bar <- ggplot(diamonds) +
  geom_bar(
    aes(x = clarity, fill = clarity),
```

---

## 5. Mention `.by` as modern alternative in data-transform.qmd

**File:** `data-transform.qmd` (the `group_by()` section)
**Risk:** Medium — adds content rather than fixing a bug
**Why it's good:** `.by` was added in dplyr 1.1.0 (Jan 2023). The book
was finalized around the same time and missed it. A brief mention like
"As of dplyr 1.1.0, you can also use the `.by` argument for per-operation
grouping" would keep the book current. Not a code change — a prose addition.

**Study first:** Read the dplyr 1.1.0 blog post and `.by` documentation
to understand when `group_by()` is still preferred (multi-step grouped
operations, grouped mutate with window functions).

---

## 6. Note about `coord_flip()` being superseded in layers.qmd

**File:** `layers.qmd` around line 935, and `EDA.qmd` line 429
**Risk:** Medium — the existing code is pedagogically intentional
**Why it's good:** Since ggplot2 3.3.0, most geoms support native
orientation. A brief note like "In modern ggplot2, you can also create
horizontal plots by mapping the categorical variable to `y` instead of
using `coord_flip()`" would be helpful. The `coord_flip()` example itself
should stay (it teaches coordinate systems), but a note acknowledges
the modern alternative.

**Study first:** Read the ggplot2 3.3.0 changelog and understand which
geoms support orientation natively vs which still need `coord_flip()`.

---

## 7. Mention `cli` package for function error messages in functions.qmd

**File:** `functions.qmd` (the error handling section, if it exists)
**Risk:** Medium-high — scope creep potential
**Why it's good:** The functions chapter teaches `stop()` for errors.
Modern tidyverse packages use `cli::cli_abort()` for structured,
informative errors with glue-style interpolation. A brief mention of
the cli alternative would point readers toward current best practice.

**Study first:** Read the functions chapter carefully. If it doesn't
cover error handling in user functions, this may not fit. Check whether
the R Packages book (r-pkgs.org) already covers this and could be
cross-referenced instead.

---

## 8. Exercise modernization: `coord_flip()` exercise in EDA.qmd

**File:** `EDA.qmd` line 429
**Risk:** Medium — changes exercise content
**Why it's good:** Exercise 3 says "add `coord_flip()` as a new layer."
Could be reworded to: "Try both approaches for creating a horizontal
boxplot: (a) mapping the categorical variable to `y`, and (b) adding
`coord_flip()`. How do they compare?" This teaches both approaches.

**Study first:** Make sure the exercise still works pedagogically — the
point is comparing approaches, not just using one.

---

## 9. Add `separate_wider_delim()` cross-reference in data-tidy.qmd

**File:** `data-tidy.qmd`
**Risk:** Medium — may already be covered elsewhere
**Why it's good:** If the tidy data chapter still uses `separate()` (now
superseded by `separate_wider_delim()` and `separate_longer_delim()`), a
note or migration would be valuable.

**Study first:** Read the full tidy data chapter to see if it already
uses the modern functions. The book may have already been updated.

---

## 10. Suggest `list_rbind()` / `list_cbind()` in iteration.qmd

**File:** `iteration.qmd`
**Risk:** Medium — depends on what the chapter already covers
**Why it's good:** `map() |> list_rbind()` replaced `map_dfr()` in
purrr 1.0.0. If the chapter still uses `map_dfr()` or doesn't mention
`list_rbind()`, adding it would be valuable.

**Study first:** Read the full iteration chapter. R4DS 2e likely already
uses the modern pattern, but verify.

---

## Submission Strategy

1. **Start with #1 or #2** — smallest, most objective, hardest to argue with
2. **Read the existing issues and PRs** first: `gh issue list --repo hadley/r4ds`
   and `gh pr list --repo hadley/r4ds` to avoid duplicates
3. **One PR per fix** for small changes (#1-4)
4. **Open an issue first** for larger suggestions (#5-10) to get buy-in
5. **Reference the book's own guidance** in your PR description (e.g.,
   "Ch. 12 recommends if_else() over ifelse(); this applies that to Ch. 1")
6. **Keep PR descriptions short** — "Fixing typos" is explicitly welcomed
7. For significant changes, include: "I assign the copyright of this
   contribution to Hadley Wickham"
