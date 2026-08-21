---
name: tecace-dashboard-ui
description: TecAce-branded dashboard UI generator — builds information-dense admin/analytics dashboards as React code on shadcn/ui (Tailwind v4) + Chart.js, with the official TecAce design system applied (brand blue #116DFF, Pretendard/Poppins, light default + dark toggle). Includes a ready-made theme CSS and a tweakcn-importable registry file, plus five full page templates (Overview, Analytics, Sales, Customers, Operations). Use this skill whenever the user mentions a dashboard, admin panel, back-office screen, analytics/metrics UI, KPI screen, status board, "shadcn", "TecAce style", or wants any screen with tables, charts, or indicators — even if they never say the word "dashboard."
---

# TecAce Dashboard UI

Generates dashboards on shadcn/ui with the TecAce design system applied. Primary output is **React + shadcn/ui + Chart.js code**; for a quick look, a static HTML preview using the same theme CSS also works.

UI copy is **English by default** (sentence case). If the product targets Korean users, switch to the Korean tone rules in `references/tokens.md` §6.

## Bundled files

| File | What it is | When to read |
| --- | --- | --- |
| `assets/tecace-theme.css` | Complete theme (shadcn vars + @theme + 19 `.ta-*` type styles + fonts) | Always — becomes the project's globals.css |
| `assets/tecace-theme.registry.json` | Registry item for shadcn CLI / tweakcn import | When the user wants tweakcn or CLI install |
| `references/tokens.md` | TecAce→shadcn mapping rationale, full palettes, rules | When making color/type decisions |
| `references/recipes.md` | Layout skeleton, KPI card/table/dark-toggle recipes, forbidden list | Before composing any screen |
| `references/charts.md` | Chart.js theme helper, per-chart rules, extended chart types | Whenever there is at least one chart |
| `references/components.md` | Per-component TecAce styling rules for 25+ shadcn components + composition patterns | When building forms, overlays, or anything beyond KPI/chart/table |
| `templates/shared.tsx` | Shared building blocks (StatCard, ChartCard, StatusBadge, PageHeader) | With any template |
| `templates/overview.tsx` | Overview page — line, doughnut, grouped bar, table | First screen |
| `templates/analytics.tsx` | Analytics page — stacked area, funnel bar, sparkline KPIs | Traffic/conversion screens |
| `templates/sales.tsx` | Sales page — combo bar+line, cumulative area, region table | Revenue screens |
| `templates/customers.tsx` | Customers page — radar, doughnut, cohort retention matrix | User/customer screens |
| `templates/operations.tsx` | Operations page — progress bars, horizontal stacked bar timeline | Project/ops screens |
| `templates/charts-gallery.tsx` | Extended chart configs — scatter, bubble, polar, gauge, waterfall, range bar, histogram, heatmap | When a page needs a chart type not in the five pages |

## Workflow

**1. Understand the ask.** What does the dashboard show (domain, metrics, data source)? How many pages? New project or an existing Next.js/Vite app? Ask before building if unclear.

**2. Scaffold** (new project):

```bash
npx create-next-app@latest my-dashboard --ts --tailwind --app --no-src-dir
cd my-dashboard && npx shadcn@latest init -d
npm i chart.js react-chartjs-2 next-themes lucide-react
```

Then batch-install shadcn components per `references/recipes.md` §0.

**3. Apply the theme.** Replace `app/globals.css` **entirely** with `assets/tecace-theme.css` (leaving the shadcn-init default block causes duplicate variable definitions). If the project is on Tailwind v3, this file won't work as-is — the values must be converted to hsl channel format per tokens.md; confirm with the user whether upgrading to v4 is an option first.

**4. Compose screens.** Follow the skeleton in `references/recipes.md` (sidebar → header → KPI row → chart rows → table row). Pick the closest page template from `templates/` as the structural base and fill it with the user's domain data. If any chart exists, create the `lib/chart-theme.ts` helper from `references/charts.md` first — and choose chart types by data shape (charts.md §2b lists the extended set: scatter, bubble, polar, gauge, waterfall, range bar, histogram, heatmap). For forms, dialogs, filter bars, and other non-chart UI, follow the per-component rules in `references/components.md`. Dark toggle via next-themes class strategy; light is the default.

**5. Verify.** Walk the checklist in recipes.md §7. At minimum grep for: no `#3366FF` or `#2AA25F` anywhere in the code, no card with both `shadow` and `border`, no body text at `font-weight: 400`.

**6. Deliver.** Hand over the code files and state, one line each, where every file goes.

## When the user wants to tweak the theme with tweakcn

- **Visual editing**: point them to https://tweakcn.com/editor/theme → **Import** → paste only the `:root` / `.dark` blocks from `assets/tecace-theme.css`. When they bring back exported CSS, **restore the brand constants (#116DFF, Pretendard, radius 12)** and apply only the rest — tweakcn is a convenience tool; Figma remains the source of truth.
- **CLI install**: drop `tecace-theme.registry.json` into the project and run `npx shadcn@latest add ./tecace-theme.registry.json`. Note the registry cannot carry the font @imports or the `.ta-*` classes — those still come from the CSS file.

## Hard rules (summary)

- Brand blue is **#116DFF**, exactly one. `#3366FF` (a known bug in the public repo's tokens) and green `#2AA25F` (from a deprecated doc) must never appear.
- The accent palette (chart-1…7) is for data visualization only. Buttons and controls use primary/semantic colors.
- Shadows stay shallow, motion is `.15s ease` fades only, no blur (secondary-button exception).
- Body text defaults to weight 500, spacing on the 4px grid, card radius 16 / dialog 20.
- English copy in sentence case, no ALL CAPS for emphasis, product name always written `TecAce`, no emoji in product UI. For Korean-market products, apply the Korean voice rules in tokens.md §6.
- When a value disagrees with the Figma source, Figma wins. Never round the odd-looking numbers (letter-spacing -0.0319em etc.).
