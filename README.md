---
title: "Chapter 2: Interstate Migration Analysis"
subtitle: "ACS Migration Flows · ACS State Controls · NOAA Climate Data· USDA Amenity Index"
date: "`r format(Sys.Date(), '%B %d, %Y')`"
output:
  html_document:
    toc: true
    toc_float:
      collapsed: false
    toc_depth: 3
    theme: flatly
    highlight: tango
    code_folding: show
    fig_caption: true
    df_print: paged
---

```{r global-options, include=FALSE}
knitr::opts_chunk$set(
  message   = FALSE,
  warning   = FALSE,
  fig.align = "center",
  dpi       = 150
)
```

---

# Setup

```{r setup}
# Setting working directory 
if (!requireNamespace("here", quietly = TRUE)) install.packages("here")
setwd(here::here())

source("R/00_setup.R")
```

---

# Part 1 — Data Collection {.tabset}

> Pulling data using APIs. Results are cached locally
> so re-running is fast after the first download.

## 1a · Census Migration Flows

**Source:** U.S. Census Bureau ACS 1-year state-to-state migration tables (2010–2024).


```{r pull-census-flows, cache=TRUE}
source("R/25_pull_census_flows.R")
```

```{r show-census-flows}
census_flows <- readRDS(file.path(DATA_DIR, "census_flows_49.rds"))

census_flows |>
  group_by(year) |>
  summarise(
    pairs      = n(),
    total_flow = scales::comma(sum(flow)),
    mean_flow  = round(mean(flow))
  ) |>
  knitr::kable(caption = "Annual Census ACS Migration Flows: 49 States")
```

## 1b · ACS State Economic Controls

**Source:** ACS 1-year estimates via tidycensus (2010 - 2024).
Variables: population, median household income, median rent, unemployment rate.

```{r pull-acs-controls, cache=TRUE}
source("R/26_pull_acs_controls.R")
```

```{r show-acs-controls}
acs_panel <- readRDS(file.path(DATA_DIR, "acs_controls_panel.rds"))

acs_panel |>
  group_by(year) |>
  summarise(
    states     = n(),
    med_income = scales::dollar(median(median_income, na.rm = TRUE)),
    med_rent   = scales::dollar(median(median_rent,   na.rm = TRUE)),
    med_unemp  = paste0(round(median(unemp_rate, na.rm = TRUE), 1), "%")
  ) |>
  knitr::kable(caption = "ACS Annual State Economic Controls")
```

## 1c · NOAA Climate Data

**Source:** NOAA GSOM (Global Summary of the Month) via CDO API, 1991 - 2020 (30-year normals).
Top 20 stations per state averaged to state-level normals.
Variables: temperature (annual, summer, winter), precipitation, sunshine %, humidity %.

```{r pull-noaa-climate, cache=TRUE}
source("R/27_pull_noaa_climate.R")
```

```{r show-climate}
climate <- readRDS(file.path(DATA_DIR, "climate_state_noaa.rds"))

climate |>
  select(abbr, temp_win, temp_ann, precip, sunshine, humidity) |>
  arrange(temp_win) |>
  knitr::kable(
    digits  = 1,
    caption = "NOAA Climate Normals by State (1991 - 2020, 30-year average)",
    col.names = c("Abbr","Winter Temp (°F)","Annual Temp (°F)",
                  "Precip (in)","Sunshine (%)","Humidity (%)")
  )
```

---

# Part 2 — Extended Bilateral Models (2021 - 2024) {.tabset}

**Regression Model** — Gravity model with full predictor set across six specifications (M1 - M5)
using ACS bilateral migration flows for 2021 - 2024.

## Code

```{r bilateral-models, cache=TRUE}
source("R/16b_bilateral_models_ext.R")
```

## Regression Table

```{r table-16b, echo=FALSE}
htmltools::includeHTML("output/tables/table_16b_extended_models.html")
```

## Coefficient Plot

```{r fig-16b, echo=FALSE, fig.cap="Figure 16b — Standardized Coefficients, Extended Bilateral Models (M1–M5)", out.width="100%"}
knitr::include_graphics("output/figures/fig_16b_coef_plot.png")
```

---

# Part 3 — PCA: Climate & Amenity {.tabset}

**Script 18** — Principal components analysis of climate and amenity variables.
NOAA climate normals; USDA Natural Amenity Index.

## Code

```{r pca, cache=TRUE}
source("R/18_pca_climate_amenity.R")
```

## Loadings Table

```{r table-18, echo=FALSE}
htmltools::includeHTML("output/tables/table_18_pca_loadings.html")
```

## Scree Plot

```{r fig-18a, echo=FALSE, fig.cap="Figure 18a — Scree Plot", out.width="70%"}
knitr::include_graphics("output/figures/fig_18a_scree.png")
```

## Biplot

```{r fig-18b, echo=FALSE, fig.cap="Figure 18b — PCA Biplot", out.width="90%"}
knitr::include_graphics("output/figures/fig_18b_biplot.png")
```

## PC Maps

```{r fig-18c, echo=FALSE, fig.cap="Figure 18c — PC1 and PC2 State Maps", out.width="49%", fig.show='hold'}
knitr::include_graphics("output/figures/fig_18c_pc1_map.png")
knitr::include_graphics("output/figures/fig_18c_pc2_map.png")
```

## Loading Heatmap

```{r fig-18d, echo=FALSE, fig.cap="Figure 18d — Loading Heatmap", out.width="80%"}
knitr::include_graphics("output/figures/fig_18d_loading_heatmap.png")
```

---

# Part 4 — Descriptive Statistics {.tabset}

**Script 19** — Summary statistics, top migration corridors, trends, and correlations.

## Code

```{r descriptives, cache=TRUE}
source("R/19_descriptives_bilateral.R")
```

## Summary Statistics

```{r table-19-1, echo=FALSE}
htmltools::includeHTML("output/tables/table_19_1_summary_stats.html")
```

## Top 30 Corridors

```{r table-19-2, echo=FALSE}
htmltools::includeHTML("output/tables/table_19_2_top30_corridors.html")
```

## State Flow Totals

```{r table-19-3, echo=FALSE}
htmltools::includeHTML("output/tables/table_19_3_state_flows.html")
```

## Annual Totals

```{r table-19-4, echo=FALSE}
htmltools::includeHTML("output/tables/table_19_4_annual_totals.html")
```

## Correlations

```{r table-19-5, echo=FALSE}
htmltools::includeHTML("output/tables/table_19_5_correlations.html")
```

## Figures

```{r fig-19a, echo=FALSE, fig.cap="Figure 19a — Top 30 Migration Corridors", out.width="90%"}
knitr::include_graphics("output/figures/fig_19a_top30_bar.png")
```

```{r fig-19b, echo=FALSE, fig.cap="Figure 19b — State Net Migration", out.width="80%"}
knitr::include_graphics("output/figures/fig_19b_state_net.png")
```

```{r fig-19c, echo=FALSE, fig.cap="Figure 19c — Flow Trend Over Time", out.width="75%"}
knitr::include_graphics("output/figures/fig_19c_flow_trend.png")
```

```{r fig-19d, echo=FALSE, fig.cap="Figure 19d — Climate vs Migration Scatter", out.width="90%"}
knitr::include_graphics("output/figures/fig_19d_climate_scatter.png")
```

```{r fig-19e, echo=FALSE, fig.cap="Figure 19e — Correlation Heatmap", out.width="80%"}
knitr::include_graphics("output/figures/fig_19e_corr_heatmap.png")
```

---

# Part 5 — Chord Diagrams {.tabset}

**Script 20** — Migration flow chord diagrams by region, year, net direction, and amenity quartile.

## Code

```{r chord-diagrams, cache=TRUE}
source("R/20_chord_diagrams.R")
```

## By Region

```{r fig-20a, echo=FALSE, fig.cap="Figure 20a — Migration Flows by Census Region", out.width="85%"}
knitr::include_graphics("output/figures/fig_20a_chord_region.png")
```

## Year-by-Year

```{r fig-20b, echo=FALSE, fig.cap="Figure 20b — Annual Migration Flow Panels", out.width="100%"}
knitr::include_graphics("output/figures/fig_20b_chord_yearly.png")
```

## Net Flow (Gainers vs. Losers)

```{r fig-20c, echo=FALSE, fig.cap="Figure 20c — Net Migration: Gainers vs. Losers", out.width="85%"}
knitr::include_graphics("output/figures/fig_20c_chord_net_flow.png")
```

## By Amenity Quartile

```{r fig-20d, echo=FALSE, fig.cap="Figure 20d — Migration by Amenity Quartile", out.width="85%"}
knitr::include_graphics("output/figures/fig_20d_chord_amenity.png")
```

---

# Part 6 — Period Comparison Models {.tabset}

**Script 21** — Three-period comparison (2010 - 2014, 2015 - 2019, 2021 - 2024) using
**Census ACS bilateral flows** and **ACS annual state controls** for all periods.

## Code

```{r period-models, cache=TRUE}
source("R/21_period_comparison_models.R")
```

## Regression Table

```{r table-21, echo=FALSE}
htmltools::includeHTML("output/tables/table_21_period_models.html")
```

## Period Coefficient Plot

```{r fig-21a, echo=FALSE, fig.cap="Figure 21a — Coefficients Across Three Periods", out.width="90%"}
knitr::include_graphics("output/figures/fig_21a_period_coef.png")
```

## Temporal Shift

```{r fig-21b, echo=FALSE, fig.cap="Figure 21b — Temporal Shift in Climate & Amenity Effects", out.width="90%"}
knitr::include_graphics("output/figures/fig_21b_period_interactions.png")
```

## Model Fit

```{r fig-21c, echo=FALSE, fig.cap="Figure 21c — R² Across Three Periods", out.width="65%"}
knitr::include_graphics("output/figures/fig_21c_r2_by_period.png")
```

---

# Part 7 — Period Coefficient Plots: Reduced Model {.tabset}

**Script 23** — Individual fig-16b-style coefficient plots for each period using the
reduced 9-variable model. Real ACS flows and controls throughout.

## Code

```{r period-coef-reduced, cache=TRUE}
source("R/23_period_coef_plots.R")
```

## 2010–2014

```{r fig-23a, echo=FALSE, fig.cap="Figure 23a — Coefficients 2010–2014", out.width="85%"}
knitr::include_graphics("output/figures/fig_23a_coef_2010_2014.png")
```

## 2015–2019

```{r fig-23b, echo=FALSE, fig.cap="Figure 23b — Coefficients 2015–2019", out.width="85%"}
knitr::include_graphics("output/figures/fig_23b_coef_2015_2019.png")
```

## 2021–2024

```{r fig-23c, echo=FALSE, fig.cap="Figure 23c — Coefficients 2021–2024", out.width="85%"}
knitr::include_graphics("output/figures/fig_23c_coef_2021_2024.png")
```

## All Periods Combined

```{r fig-23d, echo=FALSE, fig.cap="Figure 23d — All Three Periods Combined", out.width="100%"}
knitr::include_graphics("output/figures/fig_23d_coef_all_periods.png")
```

---

# Part 8 — Period Coefficient Plots: Full Model M5 {.tabset}

**Script 24** — Full M5 specification (38 predictors) per period, ACS flows, ACS controls, NOAA climate data.

## Code

```{r period-coef-full, cache=TRUE}
#source("R/24_period_coef_plots_full.R")
```

## 2010–2014

```{r fig-24a, echo=FALSE, fig.cap="Figure 24a — Full M5 Coefficients 2010–2014", out.width="85%"}
knitr::include_graphics("output/figures/fig_24a_coef_full_2010_2014.png")
```

## 2015–2019

```{r fig-24b, echo=FALSE, fig.cap="Figure 24b — Full M5 Coefficients 2015–2019", out.width="85%"}
knitr::include_graphics("output/figures/fig_24b_coef_full_2015_2019.png")
```

## 2021–2024

```{r fig-24c, echo=FALSE, fig.cap="Figure 24c — Full M5 Coefficients 2021–2024", out.width="85%"}
knitr::include_graphics("output/figures/fig_24c_coef_full_2021_2024.png")
```

---

# Part 9 — Codebook & Data Export {.tabset}

**Script 22** — Variable codebook (HTML + Word) and Excel workbook with three sheets:
flows data, state summary, and codebook.

## Code

```{r codebook-export, cache=TRUE}
source("R/22_codebook_and_export.R")
```

## Codebook

```{r table-22, echo=FALSE}
htmltools::includeHTML("output/tables/table_22_codebook.html")
```

## Excel Export

The Excel workbook `data_export_flows_ext.xlsx` has been saved to `output/tables/` with three sheets:

| Sheet | Contents |
|-------|----------|
| **flows_ext** | 5,154 bilateral flow observations × 87 variables (2021–2024) |
| **state_summary** | 49-state snapshot with real NOAA climate + ACS attributes |
| **codebook** | 56 variables documented with source and description |

---

# Session Info

```{r session-info, echo=FALSE}
sessionInfo()
```
