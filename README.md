# Chess Win-Probability Modelling

A logistic regression analysis of online chess game outcomes, built from game
data scraped from the [chess.com public API](https://www.chess.com/news/view/published-data-api).
The project fits a Bernoulli logistic model for the probability that a player
wins an individual game, and applies the fitted model to estimate a team's
winning chances against Women's World Chess Champion Ju Wenjun.

## Overview

Each observation is a single chess game with a binary outcome (win or loss).
The model predicts the log-odds of winning as a function of game-level
attributes including the Elo rating difference, playing colour, game length,
time control, opening move, and player identity.

Model building proceeds in two stages:

1. **LASSO screening** of a large candidate pool (main effects, quadratic
   terms, and all pairwise interactions) to a smaller active set.
2. **BIC-based bidirectional stepwise selection** on the screened set to
   produce a parsimonious, unpenalised final model with valid coefficient
   estimates.

Model adequacy is checked with randomised PIT (probability integral transform)
residuals, the appropriate diagnostic for a discrete Bernoulli response, along
with leverage and Cook's distance influence diagnostics.

## Data

Game data for four titled chess.com players spanning 2024–2026 is retrieved
from the public API, parsed from PGN records, and combined into a single
dataset.

| Player           | Username            | Title | Country    |
|------------------|---------------------|-------|------------|
| Mohamed Anees M  | THE_TITAN_15        | IM    | India      |
| Justin Lee       | JLL2006             | FM    | USA        |
| Simon Williams   | Ginger_GM           | GM    | UK         |
| Tokhir Nematov   | Brilliant_Move_11   | CM    | Uzbekistan |

After filtering to standard chess, removing draws and Rapid/Classical time
controls, and dropping records with incomplete Elo or move-count fields, the
modelling dataset comprises roughly 16,900 decisive games across Bullet,
Bullet+increment, Blitz, and Blitz+increment time controls.

## Method

| Stage              | Approach                                                        |
|--------------------|-----------------------------------------------------------------|
| Response           | Binary win (1) / loss (0); draws excluded                       |
| Key predictor      | `EloDiff` = player Elo − opponent Elo                           |
| Screening          | L1-penalised logistic regression, 10-fold cross-validated λ     |
| Selection          | BIC stepwise (penalty k = log n), strong hierarchy enforced     |
| Diagnostics        | Randomised PIT residuals, QQ-plot, ACF, leverage, Cook's distance |
| Application        | Predicted win probability vs. Ju Wenjun, with 95% intervals     |

## Repository structure

```
.
├── report.Rmd            # Main analysis report (R Markdown)
├── report.pdf            # Knitted report output
├── peer_assessment.tex   # Peer assessment and evaluation (LaTeX)
└── README.md             # This file
```

## Requirements

The analysis is written in R and knitted with R Markdown. It requires the
following packages:

- `rjson`, `bigchess` — data retrieval and PGN parsing
- `glmnet` — LASSO screening
- `tidyverse`, `readxl` — data manipulation and import
- `broom` — tidying model output
- `knitr`, `kableExtra` — report rendering and tables
- `ggrepel`, `forecast` — diagnostic plots

Install them in R with:

```r
install.packages(c("rjson", "tidyverse", "readxl", "glmnet",
                    "knitr", "kableExtra", "broom", "ggrepel", "forecast"))
```

The `bigchess` package may need to be installed from the CRAN archive:

```r
packageurl <- "https://cran.r-project.org/src/contrib/Archive/bigchess/bigchess_1.9.1.tar.gz"
install.packages(packageurl, repos = NULL, type = "source")
```

A LaTeX installation with the `xelatex` engine is required to knit the PDF.

## Usage

Knit the report from R:

```r
rmarkdown::render("report.Rmd")
```

Or open `report.Rmd` in RStudio and click **Knit**. The document scrapes live
data from the chess.com API on first run, so an internet connection is needed
and the initial knit may take several minutes. Scraped data is cached between
runs.

## Key findings

- The Elo difference is the dominant predictor, with each rating point
  raising the odds of winning, consistent with Elo theory.
- A measurable first-move advantage is confirmed: playing White raises the
  odds of winning by roughly 17% after conditioning on rating.
- Game length, time control, and player identity all contribute additional
  explanatory structure through main effects and interactions.
- Against Ju Wenjun (mean Elo ≈ 2728), the team is a clear underdog at every
  time control and colour, with predicted win probabilities ranging from
  about 17% to 46% depending on conditions.

## Authors

Hongyu Zhou, Mingzhuo Fan, Zebo Hu, Xinran Zhao

## Notes

This project was completed as a university group assignment. The model is an
educational exercise and its predictions should be read as indicative
benchmarks rather than precise forecasts. Game data belongs to chess.com and
the respective players; it is used here under the terms of the public API.
