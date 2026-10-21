# GitHub Pages Graph Template

Website link: https://uhm-campus-energy.github.io/Github-Pages-Template/

A GitHub Pages (Jekyll) site that turns a CSV file into interactive graphs and tables.

- **Summary** (`index.markdown`): total and top 10 items for a chosen period.
- **Period Comparison** (`Graphs/comparison.markdown`): monthly lines for several periods, all items combined.
- **Item Comparison** (`Graphs/item_comp.markdown`): monthly lines for several items in one period.
- Every graph has a "Download PNG with values" button, and every table has a "Download Excel" button.

## Using the template

1. Click on `Use this template`, rename to whatever name you want
2. Edit the settings in `_config.yml`: `title`, `baseurl`, `url`, `data_file`, `value_label`, `value_unit` and `start_month`.
3. Put your data in a CSV with the columns `item`, `date` (as `YYYY-M-D`) and `value`. The sample is `data/sample.csv`.
5. Edit `about.markdown` and `_includes/footer.html`.
6. Preview locally with `bundle exec jekyll serve`, or push to GitHub and enable Pages.

To add a page, copy one in `Graphs/`, then link it from `_includes/quick-links.html`. The shared chart code is in `_includes/chart-common.html`.
