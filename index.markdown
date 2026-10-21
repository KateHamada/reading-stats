---
layout: home
---

{% include quick-links.html %}
{% include table-export.html %}
{% include chart-common.html %}

<style>
  .stat-box {
    display: inline-block; margin: 1em 0; padding: 12px 20px;
    background: #eef4fb; border: 1px solid #c5d9ee; border-left: 5px solid #2a6fb0;
    border-radius: 6px;
  }
  .stat-box .stat-label { font-size: 0.85em; color: #4b5563; }
  .stat-box .stat-value { font-size: 1.6em; font-weight: bold; color: #1f2937; }
</style>

<h1>Summary for a specific period</h1>
<div>Select a period</div>
<div id="period-options"></div>
<div class="stat-box">
  <div class="stat-label" id="total-label">Total</div>
  <div class="stat-value" id="total">Loading...</div>
</div>

<h2>Top 10 items</h2>
<table id="top-table">
  <thead><tr><th>#</th><th>Item</th><th id="top-head">Total</th></tr></thead>
  <tbody></tbody>
</table>

<script>
  const totalEl = document.getElementById("total");
  document.getElementById("total-label").textContent = "Total " + axisLabel;
  document.getElementById("top-head").textContent = axisLabel;

  loadRecords().then(records => {
    const periods = [...new Set(records.map(r => r.period))].sort().reverse(); // newest first
    const totals = {}; // totals[period][item]
    records.forEach(r => {
      const t = totals[r.period] = totals[r.period] || {};
      t[r.item] = (t[r.item] || 0) + r.value;
    });

    const options = document.getElementById("period-options");
    periods.forEach((p, i) => {
      const label = document.createElement("label");
      label.style.marginRight = "1em";
      const radio = document.createElement("input");
      radio.type = "radio";
      radio.name = "period";
      radio.value = p;
      radio.checked = i === 0;
      radio.addEventListener("change", render);
      label.append(radio, " " + p);
      options.append(label);
    });

    function render() {
      const p = options.querySelector("input:checked").value;
      const entries = Object.entries(totals[p]).sort((a, b) => b[1] - a[1]);
      totalEl.textContent = fmt(entries.reduce((a, e) => a + e[1], 0));
      const body = document.querySelector("#top-table tbody");
      body.replaceChildren();
      entries.slice(0, 10).forEach(([item, v], i) => {
        body.insertAdjacentHTML("beforeend", "<tr><td>" + (i + 1) + "</td><td>" + esc(item) + "</td><td>" + fmt(v) + "</td></tr>");
      });
    }
    render();
  }).catch(err => { totalEl.textContent = "Failed to load data: " + err.message; });
</script>
