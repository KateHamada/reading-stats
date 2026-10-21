# Reading Stats

Website link:  https://katehamada.github.io/reading-stats/

A GitHub Pages (Jekyll) site that shows my reading stats, built from a CSV of the books I have read. Everything is calculated in the browser from the CSV, so updating the site only means updating the data file.

## Pages
- **Home** (`index.markdown`):
  - Books read and pages read for the current year and the previous year, side by side.
  - Two pie charts for the current year: comic/manga vs chapter books, and fiction vs nonfiction.
  - A "Previous years" table with books and pages read for each earlier year.
- **Books Over Time** (`Graphs/comparison.markdown`):
  - A line chart of cumulative books read, which rises when a book is finished and stays flat in months with none.
  - A bar chart of books read per month.
  - A **Year** dropdown picks a single calendar year or all years, and applies to both charts. A collapsible table shows the line chart's data.
  - The charts' PNG download includes the title and axis labels.
- **About** (`about.markdown`): a placeholder page to fill in.

A book counts toward the year and month of its **Date Finished**.

## Data
All stats come from `data/Raw Book Data.csv`, one row per book. Columns used:

| Column | Used for |
|---|---|
| `Date Finished` | Which year and month a book counts toward (`M/D/YY` or `M/D/YYYY`). |
| `Page Count` | Pages read. |
| `Fiction vs Nonfiction` | Fiction pie chart (`F` or `NF`). |
| `Comic/Manga vs Chapter Books` | Format pie chart (`CM` or `CB`). |

Other columns (`Name`, `Date Started`, `Rating`, `Genre(s)`, `e-book vs audio vs print`) are in the file but not shown yet. Rows with a blank or unreadable `Date Finished` are skipped.

## Updating the site
1. Add new rows to `data/Raw Book Data.csv`.
2. Commit and push to `main`. GitHub Pages rebuilds the site automatically (check the **Actions** tab if something fails).

## Folder layout
| Folder / file | Purpose |
|---|---|
| `index.markdown` | Home page. |
| `Graphs/comparison.markdown` | Books Over Time charts. |
| `data/` | The book CSV. |
| `_layouts/`, `_includes/` | Page layout, navigation links, footer and the Excel export for tables. |
| `_config.yml` | Site title and URL settings. |

## Leftover template files
This repo started from a graph template. `_includes/chart-common.html` and the `data_file`, `value_label`, `value_unit` and `start_month` settings in `_config.yml` are no longer used by any page and can be removed.

## Run locally
```
bundle install
bundle exec jekyll serve
```
Then open http://localhost:4000/reading-stats/.
