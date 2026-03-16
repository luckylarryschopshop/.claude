---
name: frontend
description: Browser UI standards, Chart.js patterns, CATEGORY_COLOURS definition,
  year-on-year overlays, forecast series, empty states, print layout, and partner
  view design rules. Load when building any browser UI, charts, or print output.
---

# Frontend Skill

## General Principles

- No build pipeline — vanilla JS or Alpine.js / HTMX preferred
- No framework unless the project specifically requires one
- All interactive state in JS variables or sessionStorage (never localStorage in sandboxed environments)
- Responsive by default — all layouts work at 768px–1440px
- Partner/visual view: min 16px text, min 44px touch targets

---

## CATEGORY_COLOURS

Define once, use everywhere. Same hex values in JS and Python.

```javascript
// app.js — single source of truth for all colour usage
const CATEGORY_COLOURS = {
  // Income
  "Salary":              "#1D9E75",
  "Freelance/Contract":  "#5DCAA5",
  "Investment Returns":  "#9FE1CB",
  "Refunds":             "#E1F5EE",
  "Other Income":        "#085041",

  // Essential
  "Rent/Mortgage":       "#185FA5",
  "Utilities":           "#378ADD",
  "Groceries":           "#85B7EB",
  "Transport":           "#B5D4F4",
  "Healthcare":          "#0C447C",
  "Insurance":           "#E6F1FB",
  "Loan Repayments":     "#042C53",
  "Childcare":           "#2C5F8A",
  "Provision":           "#7F77DD",

  // Non-essential
  "Dining Out":          "#D85A30",
  "Entertainment":       "#F0997B",
  "Shopping":            "#F5C4B3",
  "Subscriptions":       "#BA7517",
  "Travel":              "#EF9F27",
  "Personal Care":       "#FAC775",
  "Gifts":               "#D4537E",
  "Hobbies":             "#ED93B1",
  "Other":               "#888780",
};
```

```python
# Python mirror — same hex values
CATEGORY_COLOURS = {
    "Salary": "#1D9E75",
    # ... (identical to JS above)
}
```

Never reference a category colour inline — always look up from this map.
The map is the single source of truth for both chart colours and UI badges.

---

## Chart.js Patterns

Load via CDN:
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
```

**Always:**
- Wrap canvas in a div with explicit height and `position: relative`
- Use `responsive: true, maintainAspectRatio: false`
- Set height ONLY on the wrapper div, never on the canvas element
- Use `autoSkip: false, maxRotation: 45` for monthly x-axis labels

**Never:**
- Leave chart blank for empty data — always show empty state
- Fabricate data for missing periods — show gap or "No data" label
- Use Chart.js default legend — always build custom HTML legend

**Custom legend pattern:**
```html
<div class="chart-legend">
  <span><span class="legend-swatch" style="background:#1D9E75"></span>Groceries 32%</span>
  <span><span class="legend-swatch" style="background:#378ADD"></span>Transport 18%</span>
</div>
```
```javascript
plugins: { legend: { display: false } }  // always disable built-in legend
```

---

## Empty State (Mandatory)

Every chart must handle the case where no data exists for the selected period.
Never render a blank canvas. Never fabricate data.

```javascript
function renderOrEmpty(canvasId, chartConfig, data) {
  const ctx = document.getElementById(canvasId);
  if (!data || data.length === 0) {
    // Render empty state: axes + centred label
    new Chart(ctx, {
      type: chartConfig.type,
      data: { labels: [], datasets: [] },
      options: {
        ...chartConfig.options,
        plugins: {
          ...chartConfig.options.plugins,
          // Custom plugin to draw "No data for this period" centred
          emptyState: { message: "No data for this period" }
        }
      }
    });
    return;
  }
  new Chart(ctx, { ...chartConfig, data });
}
```

Empty state by chart type:
- **Donut**: grey ring + centred label
- **Bar**: empty axes + label centred on chart area
- **Line**: flat x-axis only + label

---

## Year-on-Year Overlay

When YoY toggle is enabled, add a second dataset in muted dashed style:

```javascript
function addYoYDataset(chart, priorYearData, currentYear) {
  // priorYearData: array matching current x-axis, null for missing months
  chart.data.datasets.push({
    label: `${currentYear - 1} (actual)`,
    data: priorYearData,
    borderColor: "#B4B2A9",        // muted grey
    backgroundColor: "transparent",
    borderDash: [6, 3],            // dashed
    borderWidth: 1.5,
    pointRadius: 3,
    spanGaps: false,               // never interpolate — gaps stay as gaps
  });
  chart.update();
}
```

Rules:
- Current year: solid line, full opacity, labelled "[year] (actual)"
- Prior year: dashed, muted grey, labelled "[year-1] (actual)"
- `spanGaps: false` — missing months show as a break in the line, never interpolated
- Both series labelled explicitly in the custom legend

---

## Forecast Series

Two forecast layers, always clearly distinguished from actual data:

```javascript
// Layer 1 — Linear trend (thin dashed neutral)
const linearForecastDataset = {
  label: "Trend projection",
  data: forecastPoints,          // null for past months, value for future only
  borderColor: "#B4B2A9",
  borderDash: [4, 4],
  borderWidth: 1,
  pointRadius: 3,
  spanGaps: false,
};

// Layer 2 — AI forecast (wider dashed distinct colour)
const aiForecastDataset = {
  label: "AI forecast (indicative only)",
  data: aiForecastPoints,
  borderColor: "#7F77DD",
  borderDash: [8, 4],
  borderWidth: 2,
  pointRadius: 4,
  spanGaps: false,
};
```

Ghost marker — when actual data arrives for a projected month:
```javascript
// Replace forecast point with actual, retain ghost
const ghostDataset = {
  label: "_ghost",               // underscore = hidden from legend
  data: ghostPoints,             // only for months where forecast existed
  pointBackgroundColor: "#D3D1C7",
  pointRadius: 4,
  borderWidth: 0,
  showLine: false,
  tooltip: {
    callbacks: {
      label: (ctx) => `Forecast was $${ctx.raw.forecast}, actual was $${ctx.raw.actual}`
    }
  }
};
```

Disclaimer (always visible near forecast charts):
```html
<p class="forecast-disclaimer">
  Forecast is indicative only. Past patterns may not predict future spending.
</p>
```

---

## Time Range Control (Reusable Component)

```javascript
// Initialise with presets and custom range pickers
function TimeRangeControl({ onChange, defaultRange = "6m" }) {
  const presets = ["6m", "12m", "24m", "ytd", "all"];
  // Custom: month-from / month-to pickers
  // YoY toggle: checkbox
  // On change: call onChange({ from, to, yoy: bool })
}
```

Partner view: expose only `6m` preset + "Show 12 months" toggle.
No custom date picker in partner view — simplicity over flexibility.

---

## Print Layout

Triggered by `window.print()`. Output to PDF via browser "Save as PDF".

**Chart capture at print time:**
```javascript
// print.js
function capturePrintCharts() {
  const staging = document.getElementById("print-canvas-staging");
  staging.style.display = "block";  // reveal hidden div

  // Render all print charts fresh — independent of current view state
  renderPrintDonut("print-category-chart", getCategoryData());

  // Wait for Chart.js to render, then capture
  requestAnimationFrame(() => {
    const canvas = staging.querySelector("canvas");
    const img = document.createElement("img");
    img.src = canvas.toDataURL("image/png");
    staging.replaceChild(img, canvas);
  });
}

window.addEventListener("beforeprint", capturePrintCharts);
window.addEventListener("afterprint", () => {
  document.getElementById("print-canvas-staging").style.display = "none";
});
```

**@media print rules:**
```css
@media print {
  /* Hide everything interactive */
  nav, .toggle-bar, .filters, .edit-controls,
  .import-panel, .refresh-btn, .badge, [data-no-print] {
    display: none !important;
  }

  /* Clean print surface */
  body { background: white; color: black; }

  /* Prevent section splitting */
  .print-section { page-break-inside: avoid; }
  .print-section h2 { page-break-after: avoid; }

  /* Traffic-light as text — colour unreliable in PDF */
  .status-green::after { content: " (On track)"; }
  .status-amber::after { content: " (Near limit)"; }
  .status-red::after   { content: " (Over budget)"; }
}
```

---

## Partner View Design Rules

- Read-only — no edit controls, no data entry, no configuration
- Min 16px text everywhere
- Min 44px touch targets on all interactive elements
- Traffic-light colours with text labels (never colour alone)
- Plain-English summaries generated client-side from data (no AI call)
- Maximum 3 screens — This Month, Trends, How Are We Doing?
- No tables. No raw numbers without context. No jargon.
