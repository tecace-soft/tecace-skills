# Chart.js — TecAce dashboard chart setup

Stack: `chart.js` + `react-chartjs-2`. Install:

```bash
npm i chart.js react-chartjs-2
```

## 1. Global defaults — one place only

Create a helper that reads the CSS variables and sets Chart.js defaults, and route every chart through it. The key requirement is that charts flip together with the dark toggle (Chart.js cannot resolve CSS variables itself, so resolve them at render time with `getComputedStyle`).

```ts
// lib/chart-theme.ts
import { Chart } from "chart.js";

export const CHART_SERIES = 7; // --chart-1 .. --chart-7

export function readChartTheme() {
  const css = getComputedStyle(document.documentElement);
  const v = (name: string) => css.getPropertyValue(name).trim();
  return {
    colors: Array.from({ length: CHART_SERIES }, (_, i) => v(`--chart-${i + 1}`)),
    fg: v("--foreground"),
    mutedFg: v("--muted-foreground"),
    grid: "rgba(112, 115, 124, 0.12)",       // fill family — safe in both modes
    card: v("--card"),
    border: v("--border"),
    success: v("--success"),
    destructive: v("--destructive"),
    fontSans: v("--font-sans"),
  };
}

export function applyChartDefaults() {
  const t = readChartTheme();
  Chart.defaults.font.family = t.fontSans;
  Chart.defaults.font.size = 12;              // .ta-caption-1 scale
  Chart.defaults.color = t.mutedFg;           // axis & legend text
  Chart.defaults.borderColor = t.grid;        // gridlines
  Chart.defaults.plugins.legend.labels.boxWidth = 8;
  Chart.defaults.plugins.legend.labels.boxHeight = 8;
  Chart.defaults.plugins.legend.labels.usePointStyle = true;
  Chart.defaults.plugins.tooltip.backgroundColor = t.card;
  Chart.defaults.plugins.tooltip.titleColor = t.fg;
  Chart.defaults.plugins.tooltip.bodyColor = t.mutedFg;
  Chart.defaults.plugins.tooltip.borderColor = t.border;
  Chart.defaults.plugins.tooltip.borderWidth = 1;
  Chart.defaults.plugins.tooltip.cornerRadius = 10;
  Chart.defaults.plugins.tooltip.padding = 12;
  Chart.defaults.animation = { duration: 150, easing: "easeOutQuad" }; // .15s ease — no bounce
}
```

After a dark toggle the charts must re-render. Passing `resolvedTheme` from `next-themes` as the chart component's `key` is the simplest reliable way:

```tsx
const { resolvedTheme } = useTheme();
useEffect(() => { applyChartDefaults(); }, [resolvedTheme]);
<Line key={resolvedTheme} data={...} options={...} />
```

## 2. Per-chart rules

**Common**: `maintainAspectRatio: false` + a fixed-height parent (`h-[280px]` etc.). Grid on the y-axis only (`x: { grid: { display: false } }`). Hide axis borders (`border: { display: false }`).

- **Line (trends)**: `tension: 0.3`, `borderWidth: 2`, `pointRadius: 0` (`pointHoverRadius: 4` on hover). Fill areas with series color + transparency (`colors[0] + "1F"` ≈ 12%). A single series uses chart-1 (brand blue) only.
- **Stacked area**: line chart with `fill: true` and `stacked: true` scales; fills at ~24% (`"3D"`), series in palette order.
- **Bar**: `borderRadius: 6`, `maxBarThickness: 32`. Stacked bars follow series order chart-1→2→3.
- **Combo (bar + line)**: bar = current metric in chart-1, line = comparison/rate in chart-2 on a second y-axis (`y1`, grid off). Never more than two axes.
- **Horizontal bar (funnel, rankings)**: `indexAxis: "y"`, single hue — chart-1 with decreasing opacity steps (`FF`, `D9`, `B3`, `8C`, `66`) rather than multiple hues, since the data is one measure across stages.
- **Doughnut**: `cutout: "72%"`, total in an HTML overlay (`.ta-numeric`) at the center. Slice borders `borderColor: card, borderWidth: 2`.
- **Radar**: max 2 series (chart-1 + chart-3), fills at 12%, `pointRadius: 2`. Grid uses the shared grid color; no angle line emphasis.
- **Sparkline (inside KPI cards)**: all axes/grid/legend/tooltips off, height ~40px. Color is always **chart-1** regardless of trend direction — the delta badge next to it carries the direction.
- **Cohort/retention matrix**: not a canvas chart — an HTML table whose cells get `background: color-mix(in srgb, var(--chart-1) N%, transparent)` scaled to the value. Text switches to white above ~50% intensity.
- **Progress**: shadcn `Progress` or a 8px `rounded-full` track (`bg-secondary`) with a `bg-primary` fill — no gradients.

## 2b. Extended chart types

Full working configs for all of these are in `templates/charts-gallery.tsx`. Pick by data shape, not by looks:

- **Scatter** (two continuous variables, e.g. deal size vs. cycle length): `pointRadius: 4`, `pointHoverRadius: 6`, max 2–3 segments colored chart-1/chart-3/chart-5. Both axes keep gridlines here (the only chart type where x-grid stays on).
- **Bubble** (three variables — x, y, size): radius encodes the third variable; scale it to 4–20px. One series per segment, fill at 66% (`hex + "A8"`), border solid.
- **Polar area** (cyclic categories with magnitude): fills at 70% (`"B3"`), `borderColor: card, borderWidth: 2`. Max 6 slices — palette order.
- **Gauge / half-doughnut** (single value vs. capacity): doughnut with `rotation: -90, circumference: 180, cutout: "78%"`, two data points `[value, 100-value]` — value in chart-1 (or `--success`/`--warning`/`--destructive` when the value itself is a health judgment), remainder in `--secondary`-equivalent (`rgba(112,115,124,.16)`). Value rendered as an HTML overlay (`.ta-numeric`) — never a plugin that draws text on canvas.
- **Waterfall** (how parts sum to a total — revenue bridge, headcount change): floating bars `[[0,4200],[4200,5100],...]`. This is a *meaning-color* chart: increases `--success`, decreases `--destructive`, start/end totals chart-1. `borderRadius: 4`, connect steps mentally — no connector lines needed.
- **Range bar** (min–max bands: salary bands, SLA ranges, temperature): horizontal floating bars `[[min,max]]`, single hue chart-1 at 60% with a solid marker for the median if needed.
- **Histogram** (distribution of one variable): bar chart with `categoryPercentage: 0.95, barPercentage: 1.0` so bars nearly touch; single hue chart-1; highlight one bin (e.g. the median bin) at full opacity and the rest at `"8C"` if you need emphasis.
- **Activity heatmap** (intensity over a 2-D grid: weekday × week): not a canvas chart — an HTML/CSS grid of `rounded` cells tinted `color-mix(in srgb, var(--chart-1) N%, transparent)` where N scales with the value (cap ~60%). Same technique as the cohort matrix. Add a min→max legend of 5 swatches.

## 3. Color usage rules (important)

- Series colors come from `--chart-1` upward, **in order**. No cherry-picking (keeps color meaning consistent across screens).
- **Meaning colors** (up/down, healthy/at-risk) are semantic tokens, not accents: `--success`, `--destructive`, `--warning`.
- More than 7 series: group the tail into "Other" before inventing colors.
- Don't plaster data labels on the chart; values live in tooltips and adjacent summary numbers.

## 4. KPI number formatting

- Values in Poppins (`.ta-numeric`), thousands separators. Abbreviations: 1.2K / 3.4M (or Korean 1.2만 / 3.4억 for Korean products) — one convention per project.
- Deltas as `+12.4%`, up `text-success` / down `text-destructive`, with lucide `ArrowUpRight` / `ArrowDownRight` (not ▲▼ glyphs in production React code).
