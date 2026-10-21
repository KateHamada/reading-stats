---
layout: home
title: "Books Over Time"
permalink: /Graphs/comparison.html
---

<style>
  .home .page-heading { display: none; }
  #year-buttons { margin: 0.5em 0; }
  #year-buttons button {
    margin: 0 6px 6px 0; padding: 4px 12px; cursor: pointer;
    border: 1px solid #c5d9ee; border-radius: 4px; background: #eef4fb; color: #1f2937;
  }
  #year-buttons button.active { background: #2a6fb0; border-color: #2a6fb0; color: #fff; }
</style>

{% include quick-links.html %}
{% include table-export.html %}
{% include chart-common.html %}

<h3>Cumulative books read</h3>
<div>Rises when a book is finished and stays flat in months with none.</div>
<div id="year-buttons"></div>
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

    const now = new Date();
    const nowKey = now.getFullYear() + "-" + String(now.getMonth() + 1).padStart(2, "0");
    const firstYear = Number(keys[0].slice(0, 4));
    const lastYear = Math.max(Number(keys[keys.length - 1].slice(0, 4)), now.getFullYear());
    const years = [];
    for (let yr = firstYear; yr <= lastYear; yr++) years.push(yr);

    const buttons = document.getElementById("year-buttons");
    ["All"].concat(years).forEach(v => {
      const b = document.createElement("button");
      b.type = "button";
      b.textContent = v === "All" ? "All years" : v;
      b.dataset.value = v;
      b.addEventListener("click", () => render(v));
      buttons.append(b);
    });

    function render(selected) {
      buttons.querySelectorAll("button").forEach(b => {
        b.classList.toggle("active", b.dataset.value === String(selected));
      });
      // every month in range, so empty months show as flat
      const startYear = selected === "All" ? firstYear : selected;
      const endYear = selected === "All" ? lastYear : selected;
      const labels = [], totals = [];
      let sum = 0;
      for (let yr = startYear; yr <= endYear; yr++) {
        for (let mo = 1; mo <= 12; mo++) {
          const key = yr + "-" + String(mo).padStart(2, "0");
          if (key > nowKey && key > keys[keys.length - 1]) break;
          sum += perMonth[key] || 0;
          labels.push(MONTH_NAMES[mo - 1] + " " + yr);
          totals.push(sum);
        }
      }
      draw(labels, totals);
    }

    function draw(labels, totals) {
    Plotly.react("chart", [{
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
    table.replaceChildren();
    table.insertAdjacentHTML("beforeend", "<tr><th>Month</th><th>Cumulative books</th></tr>");
    labels.forEach((l, i) => table.insertAdjacentHTML("beforeend", "<tr><td>" + l + "</td><td>" + totals[i] + "</td></tr>"));
    }

    render(now.getFullYear());
  }).catch(err => { msg.textContent = "Failed to load chart: " + err.message; });
</script>
