---
layout: home
title: "Item Comparison"
permalink: /Graphs/item_comp.html
---

<style>
  .home .page-heading { display: none; }
  #period-options label { margin-right: 1em; }
  .chip { margin: 0 6px 6px 0; padding: 2px 10px; border: 2px solid #999; border-radius: 14px; background: #fff; cursor: pointer; }
</style>

{% include quick-links.html %}
{% include table-export.html %}
{% include chart-common.html %}

<h3>Item comparison</h3>
<div>Pick a period, then type to search and add items to compare them.</div>
<div id="period-options"></div>
<input id="item-input" list="item-list" placeholder="Search for an item..." autocomplete="off">
<datalist id="item-list"></datalist>
<div id="item-chips"></div>
<div id="item-msg"></div>
<div id="item-chart"></div>
<details>
  <summary>Show data table</summary>
  <table id="item-table"></table>
</details>

<script>
  const chartEl = document.getElementById("item-chart");
  const msg = document.getElementById("item-msg");
  const input = document.getElementById("item-input");
  const chips = document.getElementById("item-chips");
  const chosen = [];
  const colors = {};

  loadRecords().then(records => {
    const items = [...new Set(records.map(r => r.item))].sort();
    const periods = Object.keys(sumByPeriod(records)).sort();
    const byItem = {};
    items.forEach(it => { byItem[it] = sumByPeriod(records, it); });

    const options = document.getElementById("period-options");
    periods.forEach((p, i) => {
      const label = document.createElement("label");
      label.style.marginRight = "1em";
      const radio = document.createElement("input");
      radio.type = "radio";
      radio.name = "period";
      radio.value = p;
      radio.checked = i === periods.length - 1;
      radio.addEventListener("change", render);
      label.append(radio, " " + p);
      options.append(label);
    });
    document.getElementById("item-list").innerHTML = items.map(it => '<option value="' + esc(it) + '">').join("");

    const period = () => options.querySelector("input:checked").value;

    function renderChips() {
      chips.replaceChildren();
      chosen.forEach(it => {
        const b = document.createElement("button");
        b.type = "button";
        b.className = "chip";
        b.style.borderColor = colors[it];
        b.title = "Remove";
        b.textContent = it + " ×";
        b.addEventListener("click", () => { chosen.splice(chosen.indexOf(it), 1); renderChips(); render(); });
        chips.append(b);
      });
    }

    function render() {
      const p = period();
      Plotly.purge(chartEl);
      const table = document.getElementById("item-table");
      table.replaceChildren();
      if (!chosen.length) { msg.textContent = "Search for an item and pick it to add it to the chart."; return; }
      const series = chosen.filter(it => byItem[it][p]).map(it => ({ name: it, vals: byItem[it][p], color: colors[it] }));
      if (!series.length) { msg.textContent = "The chosen items have no data for " + p + "."; return; }
      msg.textContent = chosen.length > series.length ? "Some chosen items have no data for " + p + "." : "";
      drawLines(chartEl, series, () => "Item comparison - " + p);
      fillMonthTable(table, series);
    }

    input.addEventListener("change", () => {
      const it = input.value.trim();
      if (items.includes(it) && !chosen.includes(it)) {
        colors[it] = PALETTE.find(c => !chosen.some(x => colors[x] === c)) || PALETTE[chosen.length % PALETTE.length];
        chosen.push(it);
        renderChips();
        render();
      }
      input.value = "";
    });
    render();
  }).catch(err => { msg.textContent = "Failed to load chart: " + err.message; });
</script>
