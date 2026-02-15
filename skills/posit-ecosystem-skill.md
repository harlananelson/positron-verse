# Posit Ecosystem Reference

> **Usage:** Load this file into Claude Code (via CLAUDE.md pointer) or paste into an LLM session when needing to understand the Posit ecosystem, find authoritative resources, or identify the right package for a task.

Complete map of the Posit ecosystem: organizations, key developers, books, blogs, and GitHub repositories. Use this to find authoritative sources and recommended tools.

---

## 1) Key People

### Core Leadership
| Person | GitHub | Role | Known For |
|--------|--------|------|-----------|
| JJ Allaire | [@jjallaire](https://github.com/jjallaire) | Founder/CEO | RStudio, Quarto |
| Hadley Wickham | [@hadley](https://github.com/hadley) | Chief Scientist | tidyverse, ggplot2, Advanced R |
| Joe Cheng | [@jcheng5](https://github.com/jcheng5) | CTO | Shiny framework |

### Tidyverse Team
| Person | GitHub | Known For |
|--------|--------|-----------|
| Jenny Bryan | [@jennybc](https://github.com/jennybc) | usethis, googledrive, Happy Git, project workflows |
| Lionel Henry | [@lionel-](https://github.com/lionel-) | rlang, tidy evaluation |
| Davis Vaughan | [@DavisVaughan](https://github.com/DavisVaughan) | vctrs, slider, clock |
| Gabor Csardi | [@gaborcsardi](https://github.com/gaborcsardi) | cli, pkgdown, igraph |
| Jim Hester | [@jimhester](https://github.com/jimhester) | devtools, glue (former) |

### Tidymodels Team
| Person | GitHub | Known For |
|--------|--------|-----------|
| Max Kuhn | [@topepo](https://github.com/topepo) | tidymodels, caret, Applied Predictive Modeling |
| Julia Silge | [@juliasilge](https://github.com/juliasilge) | tidymodels, tidytext, Text Mining with R |

### Visualization and Extensions
| Person | GitHub | Known For |
|--------|--------|-----------|
| Thomas Lin Pedersen | [@thomasp85](https://github.com/thomasp85) | gganimate, ggraph, patchwork, ggforce |
| Winston Chang | [@wch](https://github.com/wch) | Shiny, R6 |
| Carson Sievert | [@cpsievert](https://github.com/cpsievert) | plotly for R |
| Rich Iannone | [@rich-iannone](https://github.com/rich-iannone) | gt, DiagrammeR, pointblank |

### Publishing and Education
| Person | GitHub | Known For |
|--------|--------|-----------|
| Yihui Xie | [@yihui](https://github.com/yihui) | knitr, bookdown, blogdown, xaringan |
| Mine Çetinkaya-Rundel | [@mine-cetinkaya-rundel](https://github.com/mine-cetinkaya-rundel) | Data science education, OpenIntro |
| Charlotte Wickham | [@cwickham](https://github.com/cwickham) | Developer education, Quarto |
| Christophe Dervieux | [@cderv](https://github.com/cderv) | rmarkdown, Quarto |

### Infrastructure
| Person | GitHub | Known For |
|--------|--------|-----------|
| Kevin Ushey | [@kevinushey](https://github.com/kevinushey) | reticulate, renv |
| Garrick Aden-Buie | [@gadenbuie](https://github.com/gadenbuie) | Shiny, xaringanthemer |
| Wes McKinney | [@wesm](https://github.com/wesm) | pandas creator, Apache Arrow, Positron |

---

## 2) GitHub Organizations

### Primary
| Org | URL | Focus |
|-----|-----|-------|
| **posit-dev** | https://github.com/posit-dev | Modern tools: Positron IDE, py-shiny, great-tables |
| **rstudio** | https://github.com/rstudio | Established: Shiny, RStudio IDE, gt, reticulate |
| **tidyverse** | https://github.com/tidyverse | Core data science: ggplot2, dplyr, tidyr, purrr |
| **r-lib** | https://github.com/r-lib | Infrastructure: devtools, testthat, rlang, cli |
| **tidymodels** | https://github.com/tidymodels | ML framework: parsnip, recipes, tune, yardstick |
| **quarto-dev** | https://github.com/quarto-dev | Publishing: Quarto CLI, extensions |
| **mlverse** | https://github.com/mlverse | Deep learning: torch, luz |

### Related
| Org | URL | Focus |
|-----|-----|-------|
| **r-dbi** | https://github.com/r-dbi | Database interfaces: DBI, RSQLite, RPostgres |
| **r-spatial** | https://github.com/r-spatial | Spatial data: sf, stars |

---

## 3) Most Important Repositories

### By Stars (top 20)
| Repo | Stars | Purpose |
|------|-------|---------|
| tidyverse/ggplot2 | ~6,900 | Grammar of graphics |
| rstudio/shiny | ~5,600 | Interactive web apps |
| quarto-dev/quarto-cli | ~5,300 | Scientific publishing |
| tidyverse/dplyr | ~5,000 | Data manipulation grammar |
| rstudio/rstudio | ~4,900 | RStudio IDE |
| rstudio/bookdown | ~4,000 | Books with R Markdown |
| posit-dev/positron | ~4,000 | Next-gen data science IDE |
| rstudio/rmarkdown | ~3,000 | Dynamic documents |
| posit-dev/great-tables | ~2,600 | Publication tables (Python) |
| r-lib/devtools | ~2,500 | Package development tools |
| rstudio/gt | ~2,100 | Publication tables (R) |
| rstudio/blogdown | ~1,800 | Blogs with R Markdown |
| tidyverse/tidyverse | ~1,800 | Meta-package |
| rstudio/reticulate | ~1,700 | R ↔ Python interface |
| posit-dev/py-shiny | ~1,700 | Shiny for Python |
| tidyverse/rvest | ~1,500 | Web scraping |
| tidymodels/broom | ~1,500 | Tidy model summaries |
| rstudio/plumber | ~1,400 | R web APIs |
| tidyverse/tidyr | ~1,400 | Tidy messy data |
| tidyverse/purrr | ~1,400 | Functional programming |

---

## 4) Books (All Free Online)

### By Hadley Wickham
| Book | URL | Co-authors |
|------|-----|------------|
| R for Data Science (2e) | https://r4ds.hadley.nz/ | Mine Çetinkaya-Rundel, Garrett Grolemund |
| Advanced R (2e) | https://adv-r.hadley.nz/ | — |
| R Packages (2e) | https://r-pkgs.org/ | Jenny Bryan |
| ggplot2 (3e) | https://ggplot2-book.org/ | Danielle Navarro, Thomas Lin Pedersen |
| Mastering Shiny | https://mastering-shiny.org/ | — |
| Tidy Design Principles | https://design.tidyverse.org/ | — (in progress) |

### By Other Posit Authors
| Book | URL | Authors |
|------|-----|---------|
| Tidy Modeling with R | https://www.tmwr.org/ | Max Kuhn, Julia Silge |
| Feature Engineering and Selection | http://www.feat.engineering/ | Max Kuhn, Kjell Johnson |
| Happy Git and GitHub for the useR | https://happygitwithr.com/ | Jenny Bryan |
| What They Forgot to Teach You About R | https://rstats.wtf/ | Jenny Bryan, Jim Hester |
| Text Mining with R | https://www.tidytextmining.com/ | Julia Silge, David Robinson |

### Community Books (Posit-ecosystem)
| Book | URL | Authors |
|------|-----|---------|
| Statistical Inference via Data Science | https://moderndive.com/ | Chester Ismay, Albert Kim |
| Hands-On Machine Learning with R | https://bradleyboehmke.github.io/HOML/ | Bradley Boehmke, Brandon Greenwell |

---

## 5) Blogs

| Blog | URL | Focus |
|------|-----|-------|
| Posit Blog | https://posit.co/blog/ | Products, community, tutorials |
| Tidyverse Blog | https://tidyverse.org/blog/ | Package updates, ecosystem news |
| Posit AI Blog | https://blogs.rstudio.com/ai/ | TensorFlow, Keras, ML, deep learning |

---

## 6) Key Papers and Manifestos

| Document | URL | Significance |
|----------|-----|-------------|
| Welcome to the Tidyverse (JOSS) | https://joss.theoj.org/papers/10.21105/joss.01686 | Canonical citation |
| Tidy Data (Wickham 2014) | https://tidyr.tidyverse.org/articles/tidy-data.html | Foundation |
| Tidy Tools Manifesto | https://tidyverse.tidyverse.org/articles/manifesto.html | Design philosophy |
| Tidyverse Style Guide | https://style.tidyverse.org/ | Coding standard |

---

## 7) Conference and Community

| Resource | URL |
|----------|-----|
| posit::conf (annual conference) | https://posit.co/conference/ |
| Posit Community Forum | https://forum.posit.co/ |
| Conference Videos | https://posit.co/resources/videos/ |
| Cheatsheets | https://posit.co/resources/cheatsheets/ |
