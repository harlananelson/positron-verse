# ggplot2 Skill

> **Usage:** Load this file into Claude Code (via CLAUDE.md pointer) or paste into an LLM session for creating publication-quality visualizations following the grammar of graphics.

Comprehensive reference for ggplot2 and its extension ecosystem. Based on Hadley Wickham, Danielle Navarro, and Thomas Lin Pedersen's *ggplot2: Elegant Graphics for Data Analysis* (https://ggplot2-book.org/).

---

## 1) The Grammar of Graphics — Seven Components

| Component | Function | Required? |
|-----------|----------|-----------|
| **Data** | `ggplot(data)` | Yes |
| **Mapping** | `aes(x, y, color, ...)` | Yes |
| **Layers** | `geom_*()` / `stat_*()` | Yes (at least one) |
| **Scales** | `scale_*_*()` | Defaults provided |
| **Facets** | `facet_wrap()` / `facet_grid()` | Optional |
| **Coordinates** | `coord_*()` | Defaults to Cartesian |
| **Theme** | `theme_*()` + `theme()` | Defaults provided |

### Minimum Viable Plot
```r
ggplot(data, aes(x = var1, y = var2)) +
  geom_point()
```

---

## 2) Common Geoms

### One Variable
```r
# Continuous
geom_histogram(binwidth = 5)
geom_density(fill = "steelblue", alpha = 0.5)
geom_freqpoly(binwidth = 5)

# Discrete
geom_bar()
geom_bar(aes(y = after_stat(prop), group = 1))  # Proportions
```

### Two Variables (continuous x continuous)
```r
geom_point(alpha = 0.5)
geom_smooth(method = "lm", se = TRUE)
geom_smooth(method = "loess", span = 0.75)
geom_hex(bins = 30)            # For large datasets
geom_density_2d()
geom_rug()
```

### Two Variables (discrete x continuous)
```r
geom_boxplot()
geom_violin(draw_quantiles = c(0.25, 0.5, 0.75))
geom_jitter(width = 0.2, alpha = 0.5)
geom_col()                     # Pre-computed heights (not geom_bar)
geom_dotplot(binaxis = "y", stackdir = "center")
```

### Time Series
```r
geom_line()
geom_step()
geom_area(alpha = 0.3)
geom_ribbon(aes(ymin = lower, ymax = upper), alpha = 0.2)
```

### Statistical
```r
geom_smooth(method = "lm")
geom_quantile()
stat_summary(fun = mean, geom = "point", size = 3)
stat_summary(fun.data = mean_se, geom = "errorbar")
```

---

## 3) Aesthetic Mappings

### Inside aes() — Mapped to Data
```r
aes(x = var1, y = var2, color = group, size = value, shape = category,
    alpha = weight, fill = group, linetype = type)
```

### Outside aes() — Fixed Values
```r
geom_point(color = "steelblue", size = 3, alpha = 0.7, shape = 16)
```

### Common Shapes
| Code | Shape |
|------|-------|
| 16 | Filled circle |
| 17 | Filled triangle |
| 15 | Filled square |
| 21 | Circle with fill + color border |
| 1 | Open circle |

---

## 4) Scales

### Color Scales
```r
# Continuous
scale_color_viridis_c()                    # Colorblind-safe (default choice)
scale_color_gradient(low = "blue", high = "red")
scale_color_gradient2(low = "blue", mid = "white", high = "red", midpoint = 0)
scale_color_distiller(palette = "RdYlBu")

# Discrete
scale_color_viridis_d()
scale_color_brewer(palette = "Set1")
scale_color_manual(values = c("A" = "#E41A1C", "B" = "#377EB8"))
```

### Axis Scales
```r
scale_x_continuous(labels = scales::comma)
scale_x_continuous(labels = scales::percent)
scale_x_continuous(labels = scales::dollar)
scale_x_log10()
scale_x_sqrt()
scale_x_date(date_labels = "%b %Y", date_breaks = "3 months")

# Limits and breaks
scale_y_continuous(limits = c(0, 100), breaks = seq(0, 100, 20))

# Reverse axis
scale_y_reverse()
```

---

## 5) Faceting

```r
# Wrap — one variable
facet_wrap(~ variable, ncol = 3, scales = "free_y")

# Grid — two variables
facet_grid(rows = vars(var1), cols = vars(var2))
facet_grid(var1 ~ var2, scales = "free")

# Label formatting
facet_wrap(~ variable, labeller = labeller(variable = c(a = "Group A", b = "Group B")))
```

---

## 6) Labels and Annotations

```r
labs(
  title = "Main Title",
  subtitle = "Additional context",
  x = "X Axis Label (units)",
  y = "Y Axis Label (units)",
  color = "Legend Title",
  fill = "Fill Legend",
  caption = "Source: dataset name"
)

# Annotations (not mapped to data)
annotate("text", x = 5, y = 10, label = "Important point", size = 4)
annotate("rect", xmin = 2, xmax = 4, ymin = 5, ymax = 15, alpha = 0.2)
geom_hline(yintercept = 0, linetype = "dashed", color = "gray50")
geom_vline(xintercept = as.Date("2024-01-01"), linetype = "dotted")
geom_abline(slope = 1, intercept = 0, color = "gray50")  # Reference line

# Text labels
geom_text(aes(label = name), check_overlap = TRUE, size = 3)
geom_label(aes(label = name), size = 3)
ggrepel::geom_text_repel(aes(label = name))  # Non-overlapping
```

---

## 7) Themes

### Built-in Themes
```r
theme_minimal()      # Clean, no background (recommended default)
theme_classic()      # Axes only, publication-ready
theme_bw()           # White background with grid
theme_light()        # Light gray axes and grid
theme_void()         # Nothing (for maps, diagrams)
theme_dark()         # Dark background
```

### Customization
```r
theme(
  # Text
  plot.title = element_text(face = "bold", size = 14),
  axis.title = element_text(size = 12),
  axis.text = element_text(size = 10),
  legend.title = element_text(face = "bold"),

  # Position
  legend.position = "bottom",          # "top", "right", "left", "none"
  legend.position = c(0.8, 0.2),       # Inside plot

  # Background
  panel.grid.minor = element_blank(),
  panel.grid.major.x = element_blank(),

  # Spacing
  plot.margin = margin(10, 10, 10, 10)
)
```

### Creating a Custom Theme
```r
theme_project <- function(base_size = 12) {
  theme_minimal(base_size = base_size) +
    theme(
      plot.title = element_text(face = "bold", margin = margin(b = 10)),
      panel.grid.minor = element_blank(),
      legend.position = "bottom"
    )
}
```

---

## 8) Composing Plots (patchwork)

```r
library(patchwork)

# Side by side
p1 + p2

# Stacked
p1 / p2

# Complex layouts
(p1 + p2) / p3

# With shared annotations
(p1 + p2 + p3) +
  plot_annotation(
    title = "Combined Title",
    subtitle = "Shared subtitle",
    tag_levels = "A"           # A, B, C labels
  ) +
  plot_layout(guides = "collect")  # Shared legend
```

---

## 9) Saving Plots

```r
ggsave(
  here("output", "figures", "plot-name.png"),
  plot = p,
  width = 8,
  height = 6,
  dpi = 300,
  bg = "white"
)

# For publication
ggsave("figure.pdf", width = 180, height = 120, units = "mm")

# For presentations
ggsave("slide.png", width = 12, height = 7, dpi = 150)
```

---

## 10) Extension Packages

| Package | Purpose | Key Function |
|---------|---------|-------------|
| **patchwork** | Compose plots | `p1 + p2`, `p1 / p2` |
| **ggrepel** | Non-overlapping text | `geom_text_repel()` |
| **gganimate** | Animated plots | `transition_*()` |
| **ggridges** | Ridgeline plots | `geom_density_ridges()` |
| **ggforce** | Extra geoms | `geom_mark_ellipse()` |
| **ggdist** | Distribution viz | `stat_halfeye()` |
| **ggraph** | Network graphs | `geom_edge_*()`, `geom_node_*()` |
| **gghighlight** | Highlight subsets | `gghighlight(condition)` |
| **ggbeeswarm** | Beeswarm plots | `geom_quasirandom()` |
| **scales** | Scale helpers | `label_comma()`, `label_percent()` |
| **see** | Bayesian viz | Integration with easystats |

---

## 11) Common Recipes

### Distribution Comparison
```r
ggplot(data, aes(x = value, fill = group)) +
  geom_density(alpha = 0.5) +
  scale_fill_viridis_d() +
  labs(title = "Distribution by Group") +
  theme_minimal()
```

### Before/After with Connected Points
```r
ggplot(data, aes(x = time, y = value, group = id)) +
  geom_line(alpha = 0.3) +
  geom_point() +
  stat_summary(aes(group = 1), fun = mean, geom = "line",
               color = "red", linewidth = 1.5) +
  theme_minimal()
```

### Coefficient Plot
```r
model |>
  broom::tidy(conf.int = TRUE) |>
  filter(term != "(Intercept)") |>
  ggplot(aes(x = estimate, y = fct_reorder(term, estimate))) +
  geom_vline(xintercept = 0, linetype = "dashed") +
  geom_pointrange(aes(xmin = conf.low, xmax = conf.high)) +
  labs(x = "Estimate", y = NULL) +
  theme_minimal()
```

---

## 12) Key References

- **ggplot2 book (3e)**: https://ggplot2-book.org/
- **R4DS visualization**: https://r4ds.hadley.nz/data-visualize
- **ggplot2 reference**: https://ggplot2.tidyverse.org/reference/
- **R Graph Gallery**: https://r-graph-gallery.com/
- **Extensions gallery**: https://exts.ggplot2.tidyverse.org/gallery/
