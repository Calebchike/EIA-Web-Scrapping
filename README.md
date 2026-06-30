# EIA Crude Oil Price Web Scraping

A web scraper built from scratch to pull weekly WTI crude oil spot prices directly from the U.S. Energy Information Administration (EIA), rather than downloading a pre-packaged dataset. The project covers the full pipeline: fetch, parse, clean, reshape, and visualize.

## Data Source

The EIA publishes weekly West Texas Intermediate (WTI) crude oil spot prices at `eia.gov`. The page itself is a redirect — it serves a meta-refresh that forwards a browser to the real data endpoint at `LeafHandler.ashx`. This scraper targets that endpoint directly, since `requests` doesn't follow meta-refresh redirects the way a browser does.

## Pipeline

**1. Fetch**
The page is requested with `requests`, using a browser-style User-Agent header to avoid being blocked outright.

**2. Parse**
`BeautifulSoup` walks the raw HTML and pulls out every `<table>` on the page. The page holds eight tables in total — navigation, layout, and one holding the real price data. The data table is identified by row count: it's by far the largest, at over 500 rows.

**3. Extract**
Every row in the data table is walked cell by cell, stripped of whitespace, and collected into a list of records. That list is loaded into a pandas DataFrame.

**4. Clean**
The raw table comes out wide and duplicated — each week's date and price sit in separate columns, with redundant "End Date" / "Value" header rows mixed into the data itself. Cleaning drops the duplicate date columns, renames the remaining columns to `Year_Month`, `Week 1` through `Week 5`, strips the leftover header rows, and sets `Year_Month` as the index.

**5. Reshape**
The cleaned table is still wide — one row per month, five week-columns across. `pandas.melt()` reshapes it into long format: one row per observation, with `Year_Month`, `Week`, and `Price` columns. Blank cells (months with only four weeks) are dropped.

**6. Visualize**
Weekly prices are plotted as a time series, showing the full sweep of WTI crude pricing from 1986 to the present — covering the 1986 price collapse, the Gulf War spike, 2008, the 2014–2016 shale glut, and 2020's volatility, among others.

## Result

- **528 rows** scraped at the monthly-row level, covering **41 years** of data (1986–2026)
- **2,430 rows** after reshaping into long format, one row per week observed
- Final dataset: `Year_Month`, `Week`, `Price` (USD per barrel)

## Tools Used

- `requests` — fetching the page
- `BeautifulSoup` (bs4) — parsing HTML and locating the data table
- `pandas` — cleaning, reshaping, and structuring the data
- `matplotlib` / `seaborn` — visualization

## Why This Project

Most portfolio EDA projects start from a Kaggle CSV someone else already cleaned. This one starts from a live government webpage, with all the friction that involves — redirects, multiple candidate tables, duplicated headers, irregular row lengths. The scraping and cleaning steps are as much the point of this project as the analysis that follows.

## Notebook

The full process, including the redirect discovery, table inspection, cleaning logic, and reshaping, is documented step by step in `EIA_data_Scrapping.ipynb`.
