---
layout: home
title: "Period Comparison"
permalink: /Graphs/comparison.html
---

<style>
  .home .page-heading { display: none; }
  #period-options label { margin-right: 1em; }
</style>

{% include quick-links.html %}
{% include table-export.html %}
{% include chart-common.html %}

<h3>Total by period</h3>
<table id="totals-table">
  <thead><tr><th>Period</th><th id="totals-head">Total</th></tr></thead>
  <tbody></tbody>
</table>

<h3>Monthly values, all items combined</h3>
<div>Check the periods to compare.</div>
<div id="period-options"></div>
<div id="chart-msg"></div>
<div id="chart"></div>
<details>
  <summary>Show data table</summary>
  <table id="chart-table"></table>
</details>

<script>
  const chartEl = document.getElementById("chart");
  const msg = document.getElementById("chart-msg");
  document.getElementById("totals-head").textContent = "Total " + axisLabel;

  loadRecords().then(records => {
    const sums = sumByPeriod(records);
    const periods = Object.keys(sums).sort();
    const colorFor = p => PALETTE[periods.indexOf(p) % PALETTE.length];

    const body = document.querySelector("#totals-table tbody");
    periods.forEach(p => {
      const total = sums[p].reduce((a, v) => a + (v || 0), 0);
      body.insertAdjacentHTML("beforeend", "<tr><td>" + esc(p) + "</td><td>" + fmt(total) + "</td></tr>");
    });

    const options = document.getElementById("period-options");
    periods.forEach((p, i) => {
      const label = document.createElement("label");
      const input = document.createElement("input");
      input.type = "checkbox";
      input.value = p;
      input.checked = i >= periods.length - 2; // start with the two newest periods
      input.addEventListener("change", render);
      label.append(input, " " + p);
      options.append(label);
    });

    function render() {
      const chosen = [...options.querySelectorAll("input:checked")].map(i => i.value).sort();
      if (!chosen.length) {
        Plotly.purge(chartEl);
        document.getElementById("chart-table").replaceChildren();
        msg.textContent = "Select at least one period.";
        return;
      }
      msg.textContent = "";
      const series = chosen.map(p => ({ name: p, vals: sums[p], color: colorFor(p) }));
      drawLines(chartEl, series, () => "Monthly " + CFG.valueLabel + " by period");
      fillMonthTable(document.getElementById("chart-table"), series);
    }
    render();
  }).catch(err => { msg.textContent = "Failed to load chart: " + err.message; });
</script>
