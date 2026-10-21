---
layout: home
---

{% include quick-links.html %}
{% include table-export.html %}

<style>
  .stat-box {
    display: inline-block; margin: 1em 0; padding: 12px 20px;
    background: #eef4fb; border: 1px solid #c5d9ee; border-left: 5px solid #2a6fb0;
    border-radius: 6px;
  }
  .stat-box .stat-label { font-size: 0.85em; color: #4b5563; }
  .stat-box .stat-value { font-size: 1.6em; font-weight: bold; color: #1f2937; }
  .year-row { display: flex; flex-wrap: wrap; align-items: center; gap: 16px; }
  .year-row .stat-box { margin: 0; }
  .pie { width: 280px; height: 240px; }
</style>
<script src="https://cdn.jsdelivr.net/npm/plotly.js-dist-min@2.35.2/plotly.min.js"></script>

<h1 id="year-heading">Reading this year</h1>
<div class="year-row">
  <div class="stat-box">
    <div class="stat-label">Books read</div>
    <div class="stat-value" id="year-books">Loading...</div>
  </div>
  <div class="stat-box">
    <div class="stat-label">Pages read</div>
    <div class="stat-value" id="year-pages">Loading...</div>
  </div>
  <div class="pie" id="pie-format"></div>
  <div class="pie" id="pie-fiction"></div>
</div>
<script>
  (function () {
    const fmt = v => v.toLocaleString();
    // Minimal CSV parser that handles quoted fields containing commas.
    function parseCSV(text) {
      const rows = [];
      let row = [], field = "", inQuotes = false;
      for (let i = 0; i < text.length; i++) {
        const c = text[i];
        if (inQuotes) {
          if (c === '"' && text[i + 1] === '"') { field += '"'; i++; }
          else if (c === '"') inQuotes = false;
          else field += c;
        } else if (c === '"') inQuotes = true;
        else if (c === ",") { row.push(field); field = ""; }
        else if (c === "\n" || c === "\r") {
          if (c === "\r" && text[i + 1] === "\n") i++;
          row.push(field); field = "";
          rows.push(row); row = [];
        } else field += c;
      }
      if (field !== "" || row.length) { row.push(field); rows.push(row); }
      return rows;
    }

    const year = new Date().getFullYear();
    document.getElementById("year-heading").textContent = "Reading in " + year;
    const url = "{{ '/data/Raw Book Data.csv' | relative_url }}".replace(/ /g, "%20");
    fetch(url).then(res => {
      if (!res.ok) throw new Error("could not load book data");
      return res.text();
    }).then(text => {
      const rows = parseCSV(text);
      const header = rows[0].map(h => h.trim());
      const fi = header.indexOf("Date Finished"), pi = header.indexOf("Page Count");
      if (fi < 0 || pi < 0) throw new Error("missing Date Finished or Page Count column");
      const fiI = header.indexOf("Fiction vs Nonfiction"), fmI = header.indexOf("Comic/Manga vs Chapter Books");
      const byYear = {};
      const cur_ = { fiction: { F: 0, NF: 0 }, format: { CM: 0, CB: 0 } };
      rows.slice(1).forEach(r => {
        const m = (r[fi] || "").trim().match(/^(\d{1,2})\/(\d{1,2})\/(\d{2}|\d{4})$/);
        if (!m) return;
        const y = m[3].length === 2 ? 2000 + Number(m[3]) : Number(m[3]);
        const s = byYear[y] = byYear[y] || { books: 0, pages: 0 };
        s.books++;
        s.pages += parseInt((r[pi] || "").replace(/,/g, ""), 10) || 0;
        if (y === year) {
          const f = (r[fiI] || "").trim().toUpperCase(), c = (r[fmI] || "").trim().toUpperCase();
          if (f in cur_.fiction) cur_.fiction[f]++;
          if (c in cur_.format) cur_.format[c]++;
        }
      });
      const cur = byYear[year] || { books: 0, pages: 0 };
      document.getElementById("year-books").textContent = fmt(cur.books);
      document.getElementById("year-pages").textContent = fmt(cur.pages);

      function pie(id, title, labels, values) {
        Plotly.newPlot(id, [{
          type: "pie", labels, values, sort: false, textinfo: "value+percent",
          marker: { colors: ["#3b82c4", "#e08a2e"] }, hovertemplate: "%{label}: %{value} books<extra></extra>"
        }], {
          title: { text: title, font: { size: 14 } },
          margin: { t: 40, r: 10, b: 10, l: 10 },
          legend: { orientation: "h", y: -0.05 },
          paper_bgcolor: "rgba(0,0,0,0)", font: { color: "#1f2937" }
        }, { displaylogo: false, displayModeBar: false, responsive: true });
      }
      if (cur.books) {
        pie("pie-format", year + ": comic/manga vs chapter books", ["Comic/Manga", "Chapter books"], [cur_.format.CM, cur_.format.CB]);
        pie("pie-fiction", year + ": fiction vs nonfiction", ["Fiction", "Nonfiction"], [cur_.fiction.F, cur_.fiction.NF]);
      } else {
        document.getElementById("pie-format").textContent = "No books finished in " + year + " yet.";
      }
      const body = document.querySelector("#prev-table tbody");
      Object.keys(byYear).map(Number).filter(y => y !== year).sort((a, b) => b - a).forEach(y => {
        body.insertAdjacentHTML("beforeend", "<tr><td>" + y + "</td><td>" + fmt(byYear[y].books) + "</td><td>" + fmt(byYear[y].pages) + "</td></tr>");
      });
      if (!body.children.length) {
        body.insertAdjacentHTML("beforeend", "<tr><td colspan='3'>No previous years yet.</td></tr>");
      }
    }).catch(err => {
      document.getElementById("year-books").textContent = "Failed to load data: " + err.message;
      document.getElementById("year-pages").textContent = "";
    });
  })();
</script>

<h2>Previous years</h2>
<table id="prev-table">
  <thead><tr><th>Year</th><th>Books read</th><th>Pages read</th></tr></thead>
  <tbody></tbody>
</table>

