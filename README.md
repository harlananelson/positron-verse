# positron-verse

R best practices reference for Claude Code, defaulting to Posit/tidyverse conventions.

## Purpose

Instruct Claude Code to write R code following the patterns promoted by Posit (formerly RStudio) — tidyverse, tidymodels, ggplot2, and the broader ecosystem. Add this project as a reference in any R project's `CLAUDE.md`:

```
Use the standards as listed in positron-verse when writing R code.
```

## Structure

```
positron-verse/
├── CLAUDE.md                          # Top-level directives and invariants
├── .claude/rules/                     # Auto-loaded rules (style, design, anti-patterns)
│   ├── style-conventions.md           # Tidyverse style guide essentials
│   ├── design-principles.md           # Tidy design philosophy
│   ├── method-decision-tree.md        # Package selection guidance
│   ├── anti-patterns.md               # What to avoid
│   ├── domain-constraints.md          # R/package version constraints
│   └── output-conventions.md          # Script, plot, table, and file standards
├── skills/                            # Detailed pattern references
│   ├── tidyverse-skill.md             # dplyr, tidyr, purrr, readr, stringr, forcats
│   ├── tidymodels-skill.md            # recipes, parsnip, workflows, tune, yardstick
│   ├── ggplot2-skill.md               # Grammar of graphics, themes, extensions
│   ├── package-dev-skill.md           # devtools, roxygen2, testthat, pkgdown
│   ├── functional-programming-skill.md # map, reduce, safely, function factories
│   └── posit-ecosystem-skill.md       # People, orgs, repos, books, blogs
├── guides/                            # Reference material
│   ├── posit-books.md                 # Free online books with URLs
│   ├── posit-developers.md            # Key developers and GitHub profiles
│   └── posit-github-orgs.md           # GitHub organizations and top repos
└── examples/                          # Before/after correction snippets
    ├── corrections-from-scd-project.md
    └── corrections-from-scd-infrastructure.md
```

## Key Conventions

| Convention | Rule |
|-----------|------|
| `<-` not `=` | Assignment operator |
| `\|>` not `%>%` | Native pipe (R 4.1+) |
| `snake_case` | All names — functions, variables, columns, targets |
| `TRUE`/`FALSE` | Never `T`/`F` |
| `here::here()` | File paths |
| `if_else()` not `ifelse()` | Type-stable conditionals |
| `cli::cli_abort()` not `stop()` | Informative errors |
| `.default` not `TRUE ~` | `case_when()` fallback |
| `\(x)` not `~.x` | Lambda syntax |
| `library()` not `pacman::p_load()` | Explicit package loading |

## Sources

Built from the authoritative Posit references:

- [R for Data Science (2e)](https://r4ds.hadley.nz/) — Wickham, Cetinkaya-Rundel, Grolemund
- [Advanced R (2e)](https://adv-r.hadley.nz/) — Wickham
- [R Packages (2e)](https://r-pkgs.org/) — Wickham, Bryan
- [Tidyverse Style Guide](https://style.tidyverse.org/)
- [Tidy Design Principles](https://design.tidyverse.org/)
- [Tidy Modeling with R](https://www.tmwr.org/) — Kuhn, Silge
- [ggplot2 (3e)](https://ggplot2-book.org/) — Wickham, Navarro, Pedersen
- [Mastering Shiny](https://mastering-shiny.org/) — Wickham
