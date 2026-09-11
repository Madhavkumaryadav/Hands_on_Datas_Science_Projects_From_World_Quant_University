# Hands-on Data Science in the Mexican Real Estate Market

Part of the [Applied Data Science Lab](https://www.wqu.edu) curriculum from WorldQuant University. This project walks through a complete, four-part data science workflow — from raw, messy CSV files to a data-driven answer to a real research question — using a dataset of Mexican property listings.

## Table of Contents

- [Research Question](#research-question)
- [Project Structure](#project-structure)
- [Dataset](#dataset)
- [Notebook 1: Getting to Know the Data](#notebook-1-getting-to-know-the-data)
- [Notebook 2: Visualizing Housing Data](#notebook-2-visualizing-housing-data)
- [Notebook 3: Correlation and Variable Relationships](#notebook-3-correlation-and-variable-relationships)
- [Notebook 4: Price per Square Meter and Location Effects](#notebook-4-price-per-square-meter-and-location-effects)
- [Key Findings](#key-findings)
- [Requirements](#requirements)
- [Usage](#usage)
- [Repository Structure](#repository-structure)
- [Acknowledgments](#acknowledgments)

## Research Question

**Are property prices in Mexico more influenced by property size or by location?**

Each notebook builds on the last to answer this question with increasing rigor — starting with data cleaning, moving through visualization and correlation analysis, and ending with a quantitative variance decomposition.

## Project Structure

| # | Notebook | Focus |
|---|----------|-------|
| 1 | `Getting to Know the Data_part1.ipynb` | Data loading, cleaning, tidy data principles, and combining multiple sources |
| 2 | `Visualizing Housing Data_part2.ipynb` | Histograms, boxplots, scatterplots, and small multiples |
| 3 | `Correlation and Variable Relationships_part3.ipynb` | Pearson correlation, heatmaps, and segmented correlation (Simpson's Paradox) |
| 4 | `Price per Square Meter and Location Effects.ipynb` | Feature engineering, variance decomposition, and the final answer |

## Dataset

Three raw CSV files (`data/mexico-real-estate-1.csv`, `-2.csv`, `-3.csv`), each with roughly 700 property listings, but with inconsistent schemas — a deliberate design to practice real-world data wrangling:

- **File 1**: Prices stored as text with `$` and `,` (e.g. `"$67,965.56"`); some rows missing `lat`/`lon`
- **File 2**: Prices in Mexican pesos (`price_mxn`) instead of USD
- **File 3**: Latitude/longitude combined into a single `"lat-lon"` string column (e.g. `"19.52589,-99.151703"`), and state embedded in a pipe-delimited path (`place_with_parent_names`, e.g. `"|México|Jalisco|Guadalajara|"`)

After cleaning, the three files are combined into one master dataset of **1,736 listings** with the following columns:

| Column | Type | Description |
|---|---|---|
| `property_type` | string | `house` or `apartment` |
| `state` | string | Mexican state |
| `lat`, `lon` | float | Geographic coordinates |
| `area_m2` | float | Property size in square meters |
| `price_usd` | float | Listing price in USD |

The cleaned file is saved to `data/mexico-real-estate-combined-clean.csv` at the end of Notebook 1 and reused by every subsequent notebook.

## Notebook 1: Getting to Know the Data

**Goal:** Turn three inconsistent raw files into one clean, tidy, analysis-ready dataset.

- Introduces the **tidy data framework** (one variable per column, one observation per row, one value per cell) and identifies where the raw files violate it
- Applies the **five-method inspection ritual** — `.head()`, `.shape`, `.info()`, `.dtypes`, `.isnull().sum()` — to diagnose each raw file before touching it
- Builds a **method-chaining** cleaning pipeline using `.dropna()`, `.assign()`, and `.drop()`
- Converts currency strings to floats (stripping `$` and `,`, then `.astype(float)`)
- Converts MXN to USD using a fixed exchange rate of 19 MXN/USD
- Splits the composite `lat-lon` column with `.str.split(",", expand=True)` and extracts the state from the pipe-delimited path with `.str.split("|", expand=True)`
- Verifies schema consistency across all three cleaned DataFrames before combining
- Combines all three with `pd.concat([df1, df2, df3], ignore_index=True)`
- Produces first descriptive statistics with `.describe()`, revealing right-skewed price and area distributions (mean > median in both cases)
- Saves the final cleaned dataset to CSV for reuse in later notebooks

## Notebook 2: Visualizing Housing Data

**Goal:** Use visualization to reveal patterns that summary statistics alone cannot show.

- Establishes Matplotlib's **explicit (object-oriented) interface** as the standard: `fig, ax = plt.subplots()` → `ax.method()` → `ax.set_*()`
- **Histograms** of `price_usd` and `area_m2` confirm right-skewed distributions in both variables
- **Boxplots** compare price distributions by `property_type` and by state (Distrito Federal, Estado de México, Yucatán), introducing the five-number summary (Q1, median, Q3, whiskers, outliers)
- **Scatterplots** of area vs. price — first raw (showing compression from outliers), then after **quantile-based outlier removal** (bottom 5%, top 10%, applied to both axes)
- Adds a regression trendline with `seaborn.regplot()`, interpreting slope, confidence band width, and heteroscedasticity
- Uses `.groupby()` (split-apply-combine) to rank states by mean price
- Builds **small multiples** — one scatterplot with trendline per state — showing that the size-price relationship's *strength* (not just its price level) varies dramatically by location: strong and consistent in Yucatán, weak and noisy in Distrito Federal

## Notebook 3: Correlation and Variable Relationships

**Goal:** Replace visual impressions with a precise, quantified measure of relationship strength.

- Explains the Pearson correlation coefficient conceptually and mathematically
- Computes the **national Pearson correlation** between `area_m2` and `price_usd` (r ≈ 0.59 — moderate-to-strong, meaning area explains roughly 35% of price variance)
- Builds a full **correlation matrix** and heatmap (via `seaborn.heatmap`) across all numeric variables, identifying a `lat`-`lon` correlation that turns out to be a geographic artifact of Mexico's shape rather than a market signal
- Computes **segmented correlations by state** using a groupby → `.corr()` → `.iloc[]` → `.xs()` chain, revealing correlations ranging from near 0 to 0.9+ across different states — a textbook demonstration of **Simpson's Paradox**
- Computes segmented correlations by **property type**, finding houses (r ≈ 0.72) show a stronger size-price relationship than apartments (r ≈ 0.52)
- Engineers the **`price_per_m2`** feature (`price_usd / area_m2`) to normalize price by size
- Visualizes the "law of diminishing returns" — a curved, non-linear relationship between `area_m2` and `price_per_m2` that Pearson r understates

## Notebook 4: Price per Square Meter and Location Effects

**Goal:** Deliver a final, quantitative answer to the research question.

- Analyzes the distribution of `price_per_m2` nationally
- Aggregates and ranks states by median `price_per_m2`, filtering out groups with fewer than 5 listings to avoid small-sample artifacts
- Uses boxplots to compare the most vs. least expensive states, and apartments vs. houses, on a per-square-meter basis
- Performs a **variance decomposition**, splitting total variance in `price_usd` into between-state and within-state components, to directly quantify whether location or size better explains price
- Combines state and property type in a two-way grouped analysis to check for interaction effects

## Key Findings

- Both `price_usd` and `area_m2` are **right-skewed** — median is a more honest "typical value" than mean throughout this analysis
- The **national area-price correlation is r ≈ 0.59** — real but incomplete
- This correlation **varies enormously by state** (from near 0 to 0.9+), meaning the national average is a misleading summary of any individual local market (Simpson's Paradox)
- **Apartments have a higher `price_per_m2` than houses** (driven by urban location, amenity bundling, and diminishing returns to size)
- The **variance decomposition shows between-state variance in `price_per_m2` is larger than within-state variance** — location is the stronger overall driver of price per square meter, though size remains meaningful within any given market

## Requirements
 **->** pandas

**->** matplotlib

**->** seaborn

**->** numpy

**->** jupyter



**Install with:-**

```bash
pip install pandas matplotlib seaborn numpy jupyter
```

## Usage

1. Clone the repository and ensure the raw CSV files are placed in a `data/` folder alongside the notebooks.
2. Run the notebooks **in order (1 → 4)** — each notebook depends on outputs from the previous one, specifically the cleaned CSV produced at the end of Notebook 1 (`data/mexico-real-estate-combined-clean.csv`).
3. Each notebook contains embedded multiple-choice questions and reflection prompts for self-assessment as you work through the material.

## Repository Structure

```
DS 01 Hands-on Data Science in the Mexican Real Estate Market/
├── data/
│   ├── mexico-real-estate-1.csv
│   ├── mexico-real-estate-2.csv
│   ├── mexico-real-estate-3.csv
│   └── mexico-real-estate-combined-clean.csv   (generated by Notebook 1)
├── Getting to Know the Data_part1.ipynb
├── Visualizing Housing Data_part2.ipynb
├── Correlation and Variable Relationships_part3.ipynb
├── Price per Square Meter and Location Effects.ipynb
└── README.md
```



## Acknowledgments

Based on coursework from the **Applied Data Science Lab** at WorldQuant University.