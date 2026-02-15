# positron-verse — Claude Code Directives

**Domain:** R programming best practices following Posit conventions (tidyverse, tidymodels, ggplot2, Quarto, package development)

This extends the global `/projects/CLAUDE.md`. Reference material is in
`.claude/rules/` (auto-loaded every session). Detailed skill files are in `skills/`.

---

## 1. Work Prioritization

**Stakeholder:** Harlan (R programmer following Posit conventions)
**Task source:** Direct instructions and code review requests

### Priority Rules

1. **Posit conventions are the default.** When writing R code, follow tidyverse style guide, tidy design principles, and Posit-promoted patterns unless explicitly told otherwise.
2. **Read before write.** Verify current state before making changes.
3. **Prefer existing ecosystem solutions.** Use tidyverse/tidymodels packages before reaching for base R or external alternatives.
4. **Match the user's analysis versioning scheme.** Follow NNN-Description numbering in `/projects/CLAUDE.md`.

### Communication Style

- Reference specific skill files when explaining R patterns
- Cite the authoritative source (e.g., "per R4DS Ch. 5" or "per tidyverse style guide")
- When multiple valid approaches exist, recommend the Posit-preferred one first

---

## 2. Mode Selection

### Task Classification

- "How should I write this?" / "What's the tidyverse way?"
  → **InformationSynthesis mode**
  - Reference skills/ files and authoritative sources
  - Provide code examples following Posit patterns

- "Write this function" / "Build this pipeline" / "Create this analysis"
  → **Implementation mode**
  - Apply all rules from `.claude/rules/`
  - Follow tidyverse style guide strictly
  - Use method-decision-tree.md for tool selection

### Escalation Triggers (InformationSynthesis → Implementation)

- User asks to modify anything (create, update, delete)
- Code contradicts tidyverse style guide or design principles
- User's existing code uses anti-patterns (setwd, rm(list=ls()))

### Fail-safe: When uncertain, default to **Implementation** mode.

---

## 3. LLM Boundary

### Claude MAY

- Write R code following Posit conventions
- Suggest tidyverse alternatives to base R patterns
- Recommend packages from the Posit ecosystem
- Generate ggplot2 visualizations, tidymodels workflows, Quarto documents

### Claude SHOULD NOT

- Use `setwd()`, `rm(list = ls())`, or `library()` mid-script
- Write base R when a tidyverse equivalent exists and is clearer
- Use `=` for assignment (use `<-`)
- Use `T`/`F` instead of `TRUE`/`FALSE`
- Attach packages with `require()` in scripts (use `library()` at top)
- Use `1:length(x)` (use `seq_along(x)`)

### Claude MUST

- Follow the tidyverse style guide for all R code
- Use the native pipe `|>` (not `%>%`) unless magrittr features are needed
- Place `library()` calls at the top of scripts
- Use `here::here()` for file paths (never absolute paths or `setwd()`)
- Use snake_case for variables and functions
- Use tidy data principles (each variable a column, each observation a row)
- Document decision provenance when choosing between competing approaches

---

## 4. Workflow Convention

### Before Writing R Code

1. Check which packages are appropriate (consult method-decision-tree.md)
2. Consider whether base R or tidyverse is more appropriate for the context
3. Review the relevant skill file for the task domain

### For Significant R Code

1. Follow tidyverse style guide (rules/style-conventions.md)
2. Apply design principles (rules/design-principles.md)
3. Use appropriate tidymodels patterns for modeling tasks
4. Structure ggplot2 code with one layer per line
5. Test with representative data before finalizing

---

## 5. Key Invariants

These MUST be followed regardless of where the full explanation lives:

- **Use `|>` not `%>%`** — Native pipe is the Posit-recommended default (R ≥ 4.1)
- **Use `<-` not `=`** — Assignment operator convention per tidyverse style guide
- **snake_case everywhere** — Variables, functions, file names
- **Tidy data is the default structure** — Each variable a column, each observation a row, each value a cell
- **here::here() for paths** — Never setwd(), never absolute paths in scripts

---

## 6. Reference Files

### Auto-loaded from .claude/rules/

| File | Contents |
|------|----------|
| `style-conventions.md` | Tidyverse style guide essentials — naming, spacing, pipes, functions |
| `design-principles.md` | Tidy design principles — type stability, composability, human-centered |
| `domain-constraints.md` | R ecosystem constraints — CRAN policies, version requirements |
| `method-decision-tree.md` | When to use which package/approach for R tasks |
| `anti-patterns.md` | Common R anti-patterns to avoid and their corrections |
| `output-conventions.md` | Standard patterns for analysis outputs, reports, and visualizations |

### Skills (in skills/ directory, loaded on demand)

| File | Contents |
|------|----------|
| `tidyverse-skill.md` | Core tidyverse patterns — dplyr, tidyr, purrr, readr, stringr, forcats |
| `tidymodels-skill.md` | Machine learning workflows — recipes, parsnip, workflows, tune, yardstick |
| `ggplot2-skill.md` | Grammar of graphics — geoms, scales, facets, themes, extensions |
| `package-dev-skill.md` | R package development — devtools, usethis, testthat, roxygen2 |
| `functional-programming-skill.md` | Functional R — purrr patterns, lambda syntax, error handling |
| `posit-ecosystem-skill.md` | Complete Posit ecosystem reference — orgs, repos, books, developers |

### Guides (in guides/ directory, background reading)

| File | Contents |
|------|----------|
| `posit-books.md` | All Posit-affiliated books with URLs |
| `posit-developers.md` | Key Posit developers and their GitHub repos |
| `posit-github-orgs.md` | Posit GitHub organizations and key repositories |
