---
layout: home
title: "Books Over Time"
permalink: /Graphs/comparison.html
---

<style>
  .home .page-heading { display: none; }
</style>

{% include quick-links.html %}
{% include table-export.html %}
{% include chart-common.html %}

<h3>Cumulative books read</h3>
<div>Rises when a book is finished and stays flat in months with none.</div>
<div id="chart-msg"></div>
<div id="chart"></div>
<details>
  <summary>Show data table</summary>
  <table id="chart-table"></table>
</details>

<script>
  const msg = document.getElementById("chart-msg");
  const url = "{{ '/data/Raw Book Data.csv' | relative_url }}".replace(/ /g, "%20");

  fetch(url).then(res => {
    if (!res.ok) throw new Error("could not load book data");
    return res.text();
  }).then(text => {
    const rows = parseCSV(text);
    const fi = rows[0].map(h => h.trim()).indexOf("Date Finished");
    if (fi < 0) throw new Error("missing Date Finished column");

    const perMonth = {}; // "YYYY-MM" -> books finished
    rows.slice(1).forEach(r => {
      const m = (r[fi] || "").trim().match(/^(\d{1,2})\/\d{1,2}\/(\d{2}|\d{4})$/);
      if (!m) return;
      const y = m[2].length === 2 ? 2000 + Number(m[2]) : Number(m[2]);
      const key = y + "-" + String(m[1]).padStart(2, "0");
      perMonth[key] = (perMonth[key] || 0) + 1;
    });
    const keys = Object.keys(perMonth).sort();
    if (!keys.length) throw new Error("no finished books found");

    // every month from the first finish to now, so empty months show as flat
    const now = new Date();
    const end = now.getFullYear() + "-" + String(now.getMonth() + 1).padStart(2, "0");
    const last = keys[keys.length - 1] > end ? keys[keys.length - 1] : end;
    const labels = [], totals = [];
    let [y, mo] = keys[0].split("-").map(Number), sum = 0;
    for (;;) {
      const key = y + "-" + String(mo).padStart(2, "0");
      sum += perMonth[key] || 0;
      labels.push(MONTH_NAMES[mo - 1] + " " + y);
      totals.push(sum);
      if (key === last) break;
      if (++mo > 12) { mo = 1; y++; }
    }

    Plotly.newPlot("chart", [{
      x: labels, y: totals, type: "scatter", mode: "lines+markers",
      line: { color: PALETTE[0], width: 2 }, marker: { size: 7 },
      hovertemplate: "%{y} books"
    }], {
      height: 450,
      margin: { t: 20, r: 24, b: 60, l: 70 },
      paper_bgcolor: "rgba(0,0,0,0)", plot_bgcolor: "rgba(0,0,0,0)",
      font: { color: "#1f2937" },
      xaxis: { type: "category", gridcolor: "#d1d5db", fixedrange: true },
      yaxis: { title: "Books read (cumulative)", rangemode: "tozero", gridcolor: "#d1d5db", fixedrange: true }
    }, { responsive: true, displaylogo: false });

    const table = document.getElementById("chart-table");
    table.insertAdjacentHTML("beforeend", "<tr><th>Month</th><th>Cumulative books</th></tr>");
    labels.forEach((l, i) => table.insertAdjacentHTML("beforeend", "<tr><td>" + l + "</td><td>" + totals[i] + "</td></tr>"));
  }).catch(err => { msg.textContent = "Failed to load chart: " + err.message; });
</script>
