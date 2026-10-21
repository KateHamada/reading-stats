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
</style>

<h1 id="year-heading">Reading this year</h1>
<div class="stat-box">
  <div class="stat-label">Books read</div>
  <div class="stat-value" id="year-books">Loading...</div>
</div>
<div class="stat-box">
  <div class="stat-label">Pages read</div>
  <div class="stat-value" id="year-pages">Loading...</div>
</div>
<script>
  (function () {
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
      let books = 0, pages = 0;
      rows.slice(1).forEach(r => {
        const m = (r[fi] || "").trim().match(/^(\d{1,2})\/(\d{1,2})\/(\d{2}|\d{4})$/);
        if (!m) return;
        const y = m[3].length === 2 ? 2000 + Number(m[3]) : Number(m[3]);
        if (y !== year) return;
        books++;
        pages += parseInt((r[pi] || "").replace(/,/g, ""), 10) || 0;
      });
      document.getElementById("year-books").textContent = fmt(books);
      document.getElementById("year-pages").textContent = fmt(pages);
    }).catch(err => {
      document.getElementById("year-books").textContent = "Failed to load data: " + err.message;
      document.getElementById("year-pages").textContent = "";
    });
  })();
</script>

