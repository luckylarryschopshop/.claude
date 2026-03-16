---
name: frontend
description: Browser UI standards, chart patterns, colour map definition,
  year-on-year overlays, forecast series, empty states, print layout, and
  simplified read-only view design rules. Load when building any browser UI,
  charts, or print output.
---

# Frontend Skill

## General Principles

- No build pipeline unless the project specifically requires one
- Prefer vanilla JS or a lightweight framework (Alpine.js, HTMX)
- All interactive state in JS variables or sessionStorage
- Responsive by default — all layouts work at 768px–1440px
- Simplified/read-only views: min 16px text, min 44px touch targets

---

## Colour Map

Define once, use everywhere. The project defines its own colour map —
same values in JS and any backend language.

```javascript
// Single source of truth for all colour usage in charts and UI
const COLOUR_MAP = {
  "Category A": "#1D9E75",
  "Category B": "#185FA5",
  // ... project-specific categories
};
```

```python
# Mirror in Python — identical hex values
COLOUR_MAP = {
    "Category A": "#1D9E75",
    "Category B": "#185FA5",
}
```

Never reference a category colour inline — always look up from this map.
Used in every chart and every badge/pill throughout both views.

---

## Chart Patterns (Chart.js)

Load via CDN:
```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
```

**Always:**
- Wrap canvas in a div with explicit height and `position: relative`
- Use `responsive: true, maintainAspectRatio: false`
- Set height ONLY on the wrapper div, never on the canvas element
- Use `autoSkip: false, maxRotation: 45` for time-series x-axis labels

**Never:**
- Leave chart blank for empty data — always show an empty state
- Fabricate data for missing periods — show a gap or "No data" label
- Use the Chart.js default legend — always build a custom HTML legend

**Custom legend pattern:**
```html
<div class="chart-legend">
  <span><span class="legend-swatch" style="background:#1D9E75"></span>Category A 32%</span>
  <span><span class="legend-swatch" style="background:#185FA5"></span>Category B 18%</span>
</div>
```
```javascript
plugins: { legend: { display: false } }  // always disable built-in legend
```

---

## Empty State (Mandatory on All Charts)

Every chart must handle the case where no data exists for the selected period.
Never render a blank canvas. Never fabricate data.

```javascript
function renderOrEmpty(canvasId, chartConfig, data) {
  const ctx = document.getElementById(canvasId);
  if (!data || data.length === 0) {
    new Chart(ctx, {
      type: chartConfig.type,
      data: { labels: [], datasets: [] },
      options: {
        ...chartConfig.options,
        plugins: {
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

## Time Range Control (Reusable Component)

```javascript
// Presets and optional custom range pickers
function TimeRangeControl({ onChange, defaultRange = "6m" }) {
  const presets = ["6m", "12m", "24m", "ytd", "all"];
  // Custom: from/to month pickers
  // YoY toggle: checkbox
  // On change: call onChange({ from, to, yoy: bool })
}
```

Simplified view: expose only a default preset + one alternative toggle.
No custom date picker in simplified views — keep it focused.

---

## Year-on-Year Overlay

```javascript
function addYoYDataset(chart, priorYearData, currentYear) {
  chart.data.datasets.push({
    label: `${currentYear - 1} (actual)`,
    data: priorYearData,       // null for missing months
    borderColor: "#B4B2A9",   // muted
    backgroundColor: "transparent",
    borderDash: [6, 3],
    borderWidth: 1.5,
    pointRadius: 3,
    spanGaps: false,           // never interpolate — gaps stay as gaps
  });
  chart.update();
}
```

Rules:
- Current year: solid line, full opacity, labelled "[year] (actual)"
- Prior year: dashed, muted, labelled "[year-1] (actual)"
- `spanGaps: false` — missing months show as a break, never interpolated
- Both series explicitly labelled in the custom legend

---

## Forecast Series

Two forecast layers, clearly distinguished from actual data at all times:

```javascript
// Layer 1 — Linear trend (thin dashed neutral)
const linearForecastDataset = {
  label: "Trend projection",
  data: forecastPoints,        // null for past months, value for future only
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

Ghost marker — when actual data arrives for a projected period:
```javascript
const ghostDataset = {
  label: "_ghost",             // underscore prefix = hidden from legend
  data: ghostPoints,
  pointBackgroundColor: "#D3D1C7",
  pointRadius: 4,
  borderWidth: 0,
  showLine: false,
  tooltip: {
    callbacks: {
      label: (ctx) => `Forecast was ${ctx.raw.forecast}, actual was ${ctx.raw.actual}`
    }
  }
};
```

Disclaimer (always visible near forecast charts):
```html
<p class="forecast-disclaimer">
  Forecast is indicative only. Past patterns may not predict future results.
</p>
```

---

## Print Layout

Triggered by `window.print()`. Outputs to PDF via browser "Save as PDF".

**Chart capture at print time:**
```javascript
// Render all print charts fresh into a hidden staging div at print time
// Independent of whatever view is currently displayed
function capturePrintCharts() {
  const staging = document.getElementById("print-canvas-staging");
  staging.style.display = "block";
  // Render charts fresh here...
  requestAnimationFrame(() => {
    staging.querySelectorAll("canvas").forEach(canvas => {
      const img = document.createElement("img");
      img.src = canvas.toDataURL("image/png");
      canvas.replaceWith(img);
    });
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
  nav, .toggle-bar, .filters, .edit-controls,
  .interactive, [data-no-print] {
    display: none !important;
  }

  body { background: white; color: black; }

  .print-section { page-break-inside: avoid; }
  .print-section h2 { page-break-after: avoid; }

  /* Express status as text — colour unreliable in PDF */
  .status-green::after { content: " (On track)"; }
  .status-amber::after { content: " (Near limit)"; }
  .status-red::after   { content: " (Over budget)"; }
}
```

---

## Simplified / Read-Only View Design Rules

When building a view intended for less technical users:
- Read-only — no edit controls, no data entry, no configuration
- Min 16px text everywhere — never smaller
- Min 44px touch targets on all interactive elements
- Status communicated via colour AND text label (never colour alone)
- Plain-English summaries generated from data (avoid jargon)
- Maximum 3 screens — keep navigation minimal
- No raw data tables — use charts, large numbers, and short sentences
