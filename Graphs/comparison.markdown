---
layout: home
title: "Books Over Time"
permalink: /Graphs/comparison.html
---

<style>
  .home .page-heading { display: none; }
  #year-filter { margin: 0.5em 0; }
  #year-filter select { padding: 4px 8px; margin-left: 4px; }
</style>

{% include quick-links.html %}
{% include table-export.html %}

<h3>Cumulative books read</h3>
<div>Rises when a book is finished and stays flat in months with none.</div>
<div id="year-filter"><label>Year: <select id="year-select"></select></label></div>
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

    const buttons = document.getElementById("year-select");
    ["All"].concat(years).forEach(v => {
      const o = document.createElement("option");
      o.value = v;
      o.textContent = v === "All" ? "All years" : v;
      buttons.append(o);
    });
    buttons.addEventListener("change", () => render(buttons.value === "All" ? "All" : Number(buttons.value)));

    function render(selected) {
      buttons.value = String(selected);
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
          labels.push(selected === "All" ? MONTH_NAMES[mo - 1] + " " + yr : MONTH_NAMES[mo - 1]);
          totals.push(sum);
        }
      }
      draw(labels, totals, selected === "All" ? "Cumulative books read, all years" : "Cumulative books read in " + selected);
    }

    function draw(labels, totals, title) {
    Plotly.react("chart", [{
      x: labels, y: totals, type: "scatter", mode: "lines+markers",
      line: { color: PALETTE[0], width: 2 }, marker: { size: 7 },
      hovertemplate: "%{y} books"
    }], {
      height: 480,
      title: { text: title },
      margin: { t: 60, r: 24, b: 80, l: 70 },
      paper_bgcolor: "#ffffff", plot_bgcolor: "#ffffff",
      font: { color: "#1f2937" },
      xaxis: { title: { text: "Month" }, type: "category", gridcolor: "#d1d5db", fixedrange: true },
      yaxis: { title: { text: "Books read (cumulative)" }, rangemode: "tozero", gridcolor: "#d1d5db", fixedrange: true }
    }, {
      responsive: true, displaylogo: false,
      toImageButtonOptions: { format: "png", filename: title.replace(/[^\w]+/g, "_"), width: 1100, height: 650, scale: 2 }
    });

    const table = document.getElementById("chart-table");
    table.replaceChildren();
    table.insertAdjacentHTML("beforeend", "<tr><th>Month</th><th>Cumulative books</th></tr>");
    labels.forEach((l, i) => table.insertAdjacentHTML("beforeend", "<tr><td>" + l + "</td><td>" + totals[i] + "</td></tr>"));
    }

    render(now.getFullYear());
  }).catch(err => { msg.textContent = "Failed to load chart: " + err.message; });
</script>
