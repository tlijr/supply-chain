---
name: supply-chain
description: This skill should be used when the user asks to "map a supply chain", "show supply chain", "supply chain analysis", "supplier analysis", "who supplies [company]", "supply chain dashboard", "supply chain map", "vendor analysis", "supply chain visualization", or needs to understand the supplier/customer network of any public company. Also triggers on requests mentioning "supply chain risk", "supplier concentration", "customer concentration", "key suppliers", "key customers", or "supply chain dependencies" for any company ticker. Use this for any supply chain mapping or analysis request.
---

This skill generates a professional, interactive supply chain dashboard as a self-contained HTML document. It maps the upstream (supplier) and downstream (customer) relationships for a target company, quantifying financial interdependencies at each layer.

The output enables an analyst to understand: Who are the critical suppliers? Where is concentration risk? Which suppliers depend heavily on this company for revenue? Can I trace the chain deeper — from Tier 1 to Tier 2 suppliers?

## Output Format

The final deliverable is a **single self-contained HTML file** with:
- Embedded CSS and JavaScript (no external dependencies)
- Interactive supply chain network with clickable nodes
- Company detail cards with financials, business description, and product information
- Navigation between supply chain layers (Tier 1, Tier 2, etc.)
- Clean newspaper-style design (light mode, professional typefaces)
- All financial figures hyperlinked to Daloopa source citations
- A "Download as PDF" button (uses `window.print()`)

Write the file to `~/Generated Stuff/[TICKER]-supply-chain.html` (or user-specified location), then open it with `open`.

---

## RESEARCH WORKFLOW

This is a multi-phase research process. Each phase builds on the previous one. Maximize parallelism across independent API calls.

### Phase 1: Target Company Identification

1. Use `discover_companies` with the ticker symbol to get the `company_id`, `latest_calendar_quarter`, and `latest_fiscal_quarter`.
2. Pull key financials for the target company:
   - Use `discover_company_series` with keywords: ["revenue", "cost of goods", "gross profit", "operating income", "net income", "total cost"]
   - Use `get_company_fundamentals` for the last 4 quarters to get TTM figures
3. Note the target company's total COGS / cost of revenue (TTM) — this is the denominator for supplier % calculations.

### Phase 2: Supplier Identification

Run these concurrently to build a comprehensive supplier list:

**2a. Daloopa Document Search:**
- Search keywords: ["supplier", "vendor", "purchase", "procurement"] across last 2-4 quarters
- Search keywords: ["supply agreement", "supply chain", "manufacturing"] across last 2-4 quarters
- Search keywords: ["sole source", "single source", "key supplier"] across last 2-4 quarters
- Search keywords: ["concentration", "significant supplier"] across last 2-4 quarters
- Search the company's 10-K specifically for supplier disclosures

**2b. Web Research:**
- `"[TICKER] [company name] key suppliers list 2025 2026"` — supplier identification
- `"[TICKER] supply chain analysis suppliers"` — analyst/industry reports
- `"[TICKER] 10-K supplier disclosure"` — SEC filing analysis
- `"[company name] supply chain map"` — industry supply chain maps
- `"[company name] supplier concentration risk"` — risk analysis
- `"[company name] who manufactures for [company]"` — manufacturing partners
- `"[company name] component suppliers"` — component-level supply chain

**2c. Industry-Specific Supplier Research:**
For each industry, search for the known critical supply chain relationships:
- **Tech/Hardware**: semiconductor foundries (TSMC, Samsung), display (Samsung, LG, BOE), memory (Samsung, SK Hynix, Micron), sensors/cameras (Sony), glass (Corning), connectors (Amphenol), batteries (CATL, LG Energy), PCB/assembly (Foxconn/Hon Hai, Pegatron, Luxshare)
- **Automotive**: battery (CATL, Panasonic, LG Energy), semiconductors (Infineon, NXP, ON Semi, TI), steel (Nippon, POSCO), tires (Michelin, Bridgestone), glass (AGC, Saint-Gobain)
- **Pharma**: CDMOs (Lonza, Samsung Biologics, Catalent), API suppliers, packaging, distribution
- **Retail**: brand suppliers, logistics (FedEx, UPS), packaging
- **Energy**: equipment (Baker Hughes, Schlumberger), pipe (Tenaris), chemicals

### Phase 3: Supplier Financial Analysis

For each identified supplier (aim for 8-15 key suppliers):

1. **Discover the supplier** using `discover_companies` with their ticker
2. **Pull key financials** from Daloopa:
   - `discover_company_series` with keywords: ["revenue", "net income", "gross margin", "operating margin"]
   - `get_company_fundamentals` for last 4 quarters
3. **Determine revenue concentration**:
   - Search Daloopa documents for the supplier: keywords ["[target company name]", "customer", "concentration"]
   - Web search: `"[supplier name] [target company] revenue percentage customer"`
   - Web search: `"[supplier name] 10-K customer concentration"`
   - Many suppliers disclose their top customers in 10-K filings — look for "customers that accounted for 10% or more of revenue"
4. **Determine COGS attribution** (what % of target's costs is this supplier):
   - This is often estimated. Use logic like:
     - If Apple's COGS is ~$200B TTM and TSMC's revenue from Apple is ~$70B, then TSMC = ~35% of COGS
     - Cite the source of each estimate (analyst report, 10-K disclosure, industry research)
   - Flag when this is an estimate vs. a disclosed figure
5. **Business & product description**: What does this supplier provide? Be specific (e.g., "5nm/3nm chip fabrication for A-series and M-series SoCs" not just "semiconductors")

### Phase 4: Customer / Downstream Analysis

Research where the target company is itself a supplier:

1. **Web search**: `"[company name] major customers B2B"`, `"who does [company name] supply to"`, `"[company name] enterprise customers"`
2. **Daloopa document search**: keywords ["customer", "contract", "agreement", "enterprise", "license"]
3. For each major customer identified:
   - What product/service does the target company supply?
   - Estimated % of customer's costs attributable to the target company
   - Pull the customer's key financials from Daloopa

### Phase 5: Tier 2 Supplier Research

For the top 3-5 most important Tier 1 suppliers, repeat a lighter version of Phase 2-3:

1. Identify their key suppliers (Tier 2 to the original target)
2. Pull basic financials
3. Determine what they supply and rough revenue/cost relationships
4. This enables the "drill deeper" functionality in the dashboard

### Phase 6: Data Assembly & Synthesis

Before writing HTML, organize all data into this structure:

```
TARGET COMPANY:
  - Name, ticker, description
  - TTM Revenue, COGS, Gross Profit, Net Income, Gross Margin, Op Margin
  - Market cap, stock price (from web)

TIER 1 SUPPLIERS (sorted by estimated % of target COGS, descending):
  For each:
  - Name, ticker, description
  - What they supply (specific products/components)
  - Estimated % of target company COGS (with source/logic)
  - % of supplier revenue from target company (with source)
  - TTM Revenue, Net Income, Gross Margin
  - Market cap
  - Relationship summary (sole source? multi-source? critical?)
  - Their key suppliers (Tier 2) if researched

TIER 1 CUSTOMERS (sorted by estimated % of target revenue, descending):
  For each:
  - Name, ticker, description
  - What target company supplies to them
  - Estimated % of target revenue from this customer
  - Estimated % of customer COGS from target
  - TTM Revenue, Net Income

TIER 2 SUPPLIERS (for top 3-5 Tier 1 suppliers):
  For each Tier 1 supplier, their key suppliers with basic data
```

---

## HTML TEMPLATE & DESIGN SYSTEM

Generate the HTML following this design system. The aesthetic is "Financial Times meets clean data visualization" — light background, serif headlines, sans-serif body, minimal decoration, maximum information density.

### Design Principles
- **Light mode only** — white/off-white backgrounds
- **Newspaper typography**: Serif for headlines (Georgia, "Times New Roman"), sans-serif for body and data (system-ui, -apple-system, "Segoe UI", Helvetica)
- **Minimal color palette**: Black text, subtle gray borders, blue for links, green/red only for financial sentiment
- **Information density**: Show data compactly but with clear hierarchy
- **Interactive but not flashy**: Smooth transitions, click-to-expand, no animations for animation's sake

### Core CSS

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>[TICKER] Supply Chain Dashboard</title>
<style>
  @media print {
    body { margin: 0; }
    .no-print { display: none !important; }
    .interactive { pointer-events: none; }
    @page { margin: 0.5in; size: landscape; }
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  :root {
    --bg: #fafaf8;
    --surface: #ffffff;
    --border: #e0ddd8;
    --border-light: #f0ede8;
    --text-primary: #1a1a1a;
    --text-secondary: #555550;
    --text-tertiary: #8a8a85;
    --accent: #2563eb;
    --green: #1a7a3a;
    --green-bg: #f0f9f2;
    --red: #b91c1c;
    --red-bg: #fef2f2;
    --amber: #92600a;
    --amber-bg: #fefce8;
    --blue-bg: #eff6ff;
    --node-supplier: #e8e4df;
    --node-customer: #dde8f0;
    --node-target: #1a1a1a;
    --serif: Georgia, "Times New Roman", "Noto Serif", serif;
    --sans: system-ui, -apple-system, "Segoe UI", Helvetica, Arial, sans-serif;
    --mono: "SF Mono", "Fira Code", "Fira Mono", "Roboto Mono", monospace;
  }

  body {
    font-family: var(--sans);
    color: var(--text-primary);
    background: var(--bg);
    font-size: 14px;
    line-height: 1.55;
    -webkit-font-smoothing: antialiased;
  }

  /* Layout */
  .page-header {
    background: var(--surface);
    border-bottom: 3px solid var(--text-primary);
    padding: 24px 40px 16px;
  }
  .page-header h1 {
    font-family: var(--serif);
    font-size: 32px;
    font-weight: 700;
    letter-spacing: -0.5px;
    line-height: 1.2;
  }
  .page-header .subtitle {
    font-family: var(--sans);
    font-size: 14px;
    color: var(--text-secondary);
    margin-top: 4px;
  }
  .page-header .dateline {
    font-size: 12px;
    color: var(--text-tertiary);
    margin-top: 8px;
    font-family: var(--mono);
  }

  .container {
    max-width: 1400px;
    margin: 0 auto;
    padding: 24px 40px;
  }

  /* Section Headers */
  h2 {
    font-family: var(--serif);
    font-size: 20px;
    font-weight: 700;
    margin: 32px 0 16px;
    padding-bottom: 6px;
    border-bottom: 2px solid var(--text-primary);
    letter-spacing: -0.3px;
  }
  h3 {
    font-family: var(--sans);
    font-size: 13px;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.8px;
    color: var(--text-secondary);
    margin: 20px 0 10px;
  }

  /* KPI Bar */
  .kpi-bar {
    display: flex;
    gap: 0;
    border: 1px solid var(--border);
    border-radius: 6px;
    overflow: hidden;
    background: var(--surface);
    margin: 16px 0;
  }
  .kpi-item {
    flex: 1;
    padding: 12px 16px;
    border-right: 1px solid var(--border-light);
    text-align: center;
  }
  .kpi-item:last-child { border-right: none; }
  .kpi-label {
    font-size: 10px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    color: var(--text-tertiary);
    font-weight: 600;
  }
  .kpi-value {
    font-size: 18px;
    font-weight: 700;
    margin-top: 2px;
    font-variant-numeric: tabular-nums;
  }
  .kpi-sub {
    font-size: 11px;
    color: var(--text-tertiary);
    margin-top: 1px;
  }

  /* Supply Chain Visualization */
  .chain-view {
    display: flex;
    gap: 24px;
    align-items: flex-start;
    margin: 20px 0;
    overflow-x: auto;
    padding-bottom: 16px;
  }
  .chain-column {
    min-width: 280px;
    flex-shrink: 0;
  }
  .chain-column-header {
    font-family: var(--sans);
    font-size: 11px;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 1px;
    color: var(--text-tertiary);
    margin-bottom: 12px;
    padding-bottom: 6px;
    border-bottom: 1px solid var(--border);
    text-align: center;
  }
  .chain-arrow {
    display: flex;
    align-items: center;
    justify-content: center;
    color: var(--text-tertiary);
    font-size: 24px;
    min-width: 40px;
    padding-top: 40px;
    flex-shrink: 0;
  }

  /* Company Cards */
  .company-card {
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 14px 16px;
    margin-bottom: 10px;
    cursor: pointer;
    transition: border-color 0.15s, box-shadow 0.15s;
  }
  .company-card:hover {
    border-color: var(--text-secondary);
    box-shadow: 0 2px 8px rgba(0,0,0,0.06);
  }
  .company-card.target-card {
    background: var(--text-primary);
    color: white;
    border-color: var(--text-primary);
  }
  .company-card.target-card .card-ticker { color: rgba(255,255,255,0.7); }
  .company-card.target-card .card-metric-label { color: rgba(255,255,255,0.5); }
  .company-card.target-card .card-metric-value { color: white; }
  .company-card.target-card .card-desc { color: rgba(255,255,255,0.7); }
  .company-card.expanded { border-color: var(--accent); box-shadow: 0 2px 12px rgba(37,99,235,0.1); }
  .company-card.supplier-card { border-left: 3px solid var(--node-supplier); }
  .company-card.customer-card { border-left: 3px solid var(--node-customer); }

  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
  }
  .card-name {
    font-family: var(--serif);
    font-size: 16px;
    font-weight: 700;
    line-height: 1.2;
  }
  .card-ticker {
    font-family: var(--mono);
    font-size: 11px;
    color: var(--text-tertiary);
    margin-top: 2px;
  }
  .card-badge {
    font-size: 10px;
    font-weight: 700;
    padding: 2px 8px;
    border-radius: 3px;
    white-space: nowrap;
  }
  .badge-pct-high { background: var(--red-bg); color: var(--red); }
  .badge-pct-med { background: var(--amber-bg); color: var(--amber); }
  .badge-pct-low { background: var(--green-bg); color: var(--green); }

  .card-supplies {
    font-size: 12px;
    color: var(--text-secondary);
    margin-top: 6px;
    line-height: 1.4;
  }

  .card-metrics {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 8px;
    margin-top: 10px;
    padding-top: 10px;
    border-top: 1px solid var(--border-light);
  }
  .card-metric-label {
    font-size: 9px;
    text-transform: uppercase;
    letter-spacing: 0.3px;
    color: var(--text-tertiary);
  }
  .card-metric-value {
    font-size: 13px;
    font-weight: 700;
    font-variant-numeric: tabular-nums;
  }

  .card-desc {
    font-size: 12px;
    color: var(--text-secondary);
    margin-top: 8px;
    line-height: 1.45;
  }

  /* Expanded Detail Panel */
  .detail-panel {
    display: none;
    margin-top: 12px;
    padding-top: 12px;
    border-top: 1px solid var(--border-light);
  }
  .company-card.expanded .detail-panel { display: block; }

  .detail-section {
    margin-bottom: 14px;
  }
  .detail-section h4 {
    font-size: 11px;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    color: var(--text-tertiary);
    margin-bottom: 6px;
  }

  .relationship-bar {
    height: 8px;
    background: var(--border-light);
    border-radius: 4px;
    overflow: hidden;
    margin: 4px 0;
  }
  .relationship-fill {
    height: 100%;
    border-radius: 4px;
    transition: width 0.3s;
  }
  .fill-red { background: var(--red); }
  .fill-amber { background: var(--amber); }
  .fill-green { background: var(--green); }
  .fill-blue { background: var(--accent); }

  /* Drill-down button */
  .drill-btn {
    display: inline-block;
    font-size: 11px;
    font-weight: 600;
    color: var(--accent);
    cursor: pointer;
    padding: 4px 0;
    border: none;
    background: none;
    font-family: var(--sans);
  }
  .drill-btn:hover { text-decoration: underline; }

  /* Concentration Table */
  .conc-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 13px;
    margin: 12px 0;
  }
  .conc-table th {
    font-size: 10px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    color: var(--text-tertiary);
    font-weight: 600;
    text-align: left;
    padding: 6px 10px;
    border-bottom: 2px solid var(--border);
    background: var(--bg);
  }
  .conc-table th:not(:first-child) { text-align: right; }
  .conc-table td {
    padding: 8px 10px;
    border-bottom: 1px solid var(--border-light);
    font-variant-numeric: tabular-nums;
  }
  .conc-table td:not(:first-child) { text-align: right; }
  .conc-table tr:hover { background: var(--blue-bg); }
  .conc-table .row-total {
    font-weight: 700;
    border-top: 2px solid var(--border);
    background: var(--bg);
  }

  /* Risk indicator */
  .risk-tag {
    display: inline-block;
    font-size: 10px;
    font-weight: 700;
    padding: 1px 6px;
    border-radius: 3px;
  }
  .risk-high { background: var(--red-bg); color: var(--red); }
  .risk-med { background: var(--amber-bg); color: var(--amber); }
  .risk-low { background: var(--green-bg); color: var(--green); }

  /* Tabs for switching views */
  .tab-bar {
    display: flex;
    gap: 0;
    border-bottom: 2px solid var(--border);
    margin-bottom: 20px;
  }
  .tab {
    padding: 10px 20px;
    font-size: 13px;
    font-weight: 600;
    color: var(--text-tertiary);
    cursor: pointer;
    border-bottom: 2px solid transparent;
    margin-bottom: -2px;
    transition: color 0.15s, border-color 0.15s;
    font-family: var(--sans);
    background: none;
    border-top: none;
    border-left: none;
    border-right: none;
  }
  .tab:hover { color: var(--text-primary); }
  .tab.active {
    color: var(--text-primary);
    border-bottom-color: var(--text-primary);
  }
  .tab-content { display: none; }
  .tab-content.active { display: block; }

  /* Methodology / Source Notes */
  .methodology-box {
    background: var(--bg);
    border: 1px solid var(--border);
    border-radius: 6px;
    padding: 16px 20px;
    margin: 16px 0;
    font-size: 12px;
    color: var(--text-secondary);
    line-height: 1.5;
  }
  .methodology-box h4 {
    font-size: 11px;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    color: var(--text-tertiary);
    margin-bottom: 8px;
  }

  /* Links & References */
  a { color: var(--accent); text-decoration: none; }
  a:hover { text-decoration: underline; }

  .source-tag {
    font-size: 9px;
    color: var(--text-tertiary);
    font-style: italic;
  }

  /* Footer */
  .page-footer {
    margin-top: 40px;
    padding: 16px 0;
    border-top: 3px solid var(--text-primary);
    font-size: 11px;
    color: var(--text-tertiary);
    text-align: center;
  }

  /* Print button */
  .dl-btn {
    display: inline-block;
    padding: 10px 24px;
    background: var(--text-primary);
    color: white;
    border: none;
    border-radius: 5px;
    font-size: 13px;
    font-weight: 600;
    cursor: pointer;
    font-family: var(--sans);
  }
  .dl-btn:hover { background: #333; }

  /* Two-column layout */
  .two-col {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
  }
  @media (max-width: 800px) {
    .two-col { grid-template-columns: 1fr; }
    .chain-view { flex-direction: column; }
    .chain-arrow { transform: rotate(90deg); padding-top: 0; }
  }
</style>
</head>
```

### Core JavaScript Pattern

The dashboard uses vanilla JavaScript for interactivity. Include these functions in a `<script>` tag at the end of the body:

```javascript
<script>
// Tab switching
function switchTab(tabId) {
  document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
  document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
  document.querySelector(`[data-tab="${tabId}"]`).classList.add('active');
  document.getElementById(tabId).classList.add('active');
}

// Card expand/collapse
function toggleCard(cardId) {
  const card = document.getElementById(cardId);
  card.classList.toggle('expanded');
}

// Navigate to tier 2 view for a specific supplier
function drillDown(companyTicker) {
  // Switch to the tier-2 tab and scroll to the relevant section
  switchTab('tier2');
  const section = document.getElementById('tier2-' + companyTicker);
  if (section) {
    section.scrollIntoView({ behavior: 'smooth', block: 'start' });
    section.style.outline = '2px solid var(--accent)';
    setTimeout(() => { section.style.outline = 'none'; }, 2000);
  }
}

// Initialize
document.addEventListener('DOMContentLoaded', () => {
  // Set first tab active
  const firstTab = document.querySelector('.tab');
  if (firstTab) firstTab.click();
});
</script>
```

---

## DOCUMENT STRUCTURE

The HTML document has these sections. Every section is mandatory.

### Section 1: Print Button Bar
```html
<div class="no-print" style="text-align:center; padding:16px; background:var(--surface); border-bottom:1px solid var(--border);">
  <button class="dl-btn" onclick="window.print()">Download as PDF</button>
  <span style="font-size:12px; color:var(--text-tertiary); margin-left:12px;">or Cmd+P &rarr; Save as PDF</span>
</div>
```

### Section 2: Page Header
```html
<div class="page-header">
  <h1>[TICKER] &mdash; Supply Chain Map</h1>
  <div class="subtitle">[Full Company Name] &middot; Interactive Supply Chain Analysis</div>
  <div class="dateline">Prepared [Date] &middot; Data sourced from <a href="https://daloopa.com">Daloopa</a> &middot; TTM through [Latest Quarter]</div>
</div>
```

### Section 3: Target Company KPI Bar
Show 6 KPIs for the target company:
```html
<div class="container">
  <div class="kpi-bar">
    <div class="kpi-item"><div class="kpi-label">TTM Revenue</div><div class="kpi-value">$XXB</div></div>
    <div class="kpi-item"><div class="kpi-label">TTM COGS</div><div class="kpi-value">$XXB</div></div>
    <div class="kpi-item"><div class="kpi-label">Gross Margin</div><div class="kpi-value">XX.X%</div></div>
    <div class="kpi-item"><div class="kpi-label">Suppliers Mapped</div><div class="kpi-value">XX</div></div>
    <div class="kpi-item"><div class="kpi-label">Top 5 = % of COGS</div><div class="kpi-value">~XX%</div></div>
    <div class="kpi-item"><div class="kpi-label">Key Customers</div><div class="kpi-value">XX</div></div>
  </div>
```

### Section 4: Tab Navigation
```html
  <div class="tab-bar">
    <button class="tab active" data-tab="overview" onclick="switchTab('overview')">Supply Chain Map</button>
    <button class="tab" data-tab="suppliers" onclick="switchTab('suppliers')">Supplier Detail</button>
    <button class="tab" data-tab="customers" onclick="switchTab('customers')">Customer Detail</button>
    <button class="tab" data-tab="tier2" onclick="switchTab('tier2')">Tier 2 Suppliers</button>
    <button class="tab" data-tab="concentration" onclick="switchTab('concentration')">Concentration Analysis</button>
  </div>
```

### Section 5: Tab Content — Supply Chain Map (Overview)
This is the visual chain layout showing Tier 2 → Tier 1 → Target → Customers in a horizontal flow.

```html
<div id="overview" class="tab-content active">
  <h2>Supply Chain Overview</h2>
  <p style="color:var(--text-secondary); margin-bottom:16px;">Click any company to expand details. Use "Explore Tier 2" to drill deeper into the supply chain.</p>

  <div class="chain-view">
    <!-- Tier 2 column (optional, show top 3-5) -->
    <div class="chain-column">
      <div class="chain-column-header">Tier 2 Suppliers</div>
      <!-- Compact cards for Tier 2 -->
    </div>

    <div class="chain-arrow">&rarr;</div>

    <!-- Tier 1 Suppliers column -->
    <div class="chain-column">
      <div class="chain-column-header">Tier 1 Suppliers</div>
      <!-- Company cards for each Tier 1 supplier -->
      <div class="company-card supplier-card" id="card-[TICKER]" onclick="toggleCard('card-[TICKER]')">
        <div class="card-header">
          <div>
            <div class="card-name">[Company Name]</div>
            <div class="card-ticker">[TICKER]</div>
          </div>
          <span class="card-badge badge-pct-high">~XX% of COGS</span>
        </div>
        <div class="card-supplies">Supplies: [specific products/components]</div>
        <div class="card-metrics">
          <div><div class="card-metric-label">Revenue</div><div class="card-metric-value"><a href="https://daloopa.com/src/[id]">$XXB</a></div></div>
          <div><div class="card-metric-label">[Target] Rev %</div><div class="card-metric-value">XX%</div></div>
          <div><div class="card-metric-label">Gross Margin</div><div class="card-metric-value"><a href="https://daloopa.com/src/[id]">XX%</a></div></div>
        </div>
        <!-- Expanded detail panel -->
        <div class="detail-panel">
          <div class="detail-section">
            <h4>Business Description</h4>
            <p class="card-desc">[2-3 sentence description]</p>
          </div>
          <div class="detail-section">
            <h4>Relationship with [TARGET]</h4>
            <p class="card-desc">[What they supply, how critical, sole source?, etc.]</p>
            <div style="margin-top:8px;">
              <div style="display:flex; justify-content:space-between; font-size:11px;">
                <span>% of [TARGET] COGS</span><span class="bold">~XX%</span>
              </div>
              <div class="relationship-bar"><div class="relationship-fill fill-[color]" style="width:XX%"></div></div>
              <div style="display:flex; justify-content:space-between; font-size:11px; margin-top:6px;">
                <span>[TARGET] as % of their revenue</span><span class="bold">~XX%</span>
              </div>
              <div class="relationship-bar"><div class="relationship-fill fill-blue" style="width:XX%"></div></div>
            </div>
          </div>
          <div class="detail-section">
            <h4>Methodology</h4>
            <p class="source-tag">[Explain how the % estimates were derived, cite sources]</p>
          </div>
          <button class="drill-btn" onclick="event.stopPropagation(); drillDown('[TICKER]')">Explore Tier 2 suppliers &rarr;</button>
        </div>
      </div>
      <!-- Repeat for each Tier 1 supplier -->
    </div>

    <div class="chain-arrow">&rarr;</div>

    <!-- Target Company (center) -->
    <div class="chain-column">
      <div class="chain-column-header">Target Company</div>
      <div class="company-card target-card">
        <div class="card-name">[TARGET COMPANY]</div>
        <div class="card-ticker">[TICKER]</div>
        <div class="card-desc">[1 sentence description]</div>
        <div class="card-metrics">
          <div><div class="card-metric-label">TTM Revenue</div><div class="card-metric-value"><a href="https://daloopa.com/src/[id]" style="color:white">$XXB</a></div></div>
          <div><div class="card-metric-label">TTM COGS</div><div class="card-metric-value"><a href="https://daloopa.com/src/[id]" style="color:white">$XXB</a></div></div>
          <div><div class="card-metric-label">Gross Margin</div><div class="card-metric-value">XX.X%</div></div>
        </div>
      </div>
    </div>

    <div class="chain-arrow">&rarr;</div>

    <!-- Customers column -->
    <div class="chain-column">
      <div class="chain-column-header">Key Customers</div>
      <!-- Company cards for each major customer -->
    </div>
  </div>
</div>
```

### Section 6: Tab Content — Supplier Detail Table
A sortable summary table of all Tier 1 suppliers with full financial data.

```html
<div id="suppliers" class="tab-content">
  <h2>Tier 1 Supplier Analysis</h2>
  <table class="conc-table">
    <thead>
      <tr>
        <th>Company</th>
        <th>Supplies</th>
        <th>Est. % of COGS</th>
        <th>[TARGET] as % of Rev</th>
        <th>TTM Revenue</th>
        <th>Gross Margin</th>
        <th>TTM Net Income</th>
        <th>Concentration Risk</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><strong>[Company]</strong> <span class="card-ticker">[TICKER]</span></td>
        <td>[Products]</td>
        <td>~XX%</td>
        <td>~XX%</td>
        <td><a href="https://daloopa.com/src/[id]">$XXB</a></td>
        <td><a href="https://daloopa.com/src/[id]">XX.X%</a></td>
        <td><a href="https://daloopa.com/src/[id]">$X.XB</a></td>
        <td><span class="risk-tag risk-[high|med|low]">[HIGH|MED|LOW]</span></td>
      </tr>
      <!-- Repeat for each supplier -->
      <tr class="row-total">
        <td><strong>Top Suppliers Total</strong></td>
        <td></td>
        <td>~XX%</td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
      </tr>
    </tbody>
  </table>

  <div class="methodology-box">
    <h4>Estimation Methodology</h4>
    <p><strong>% of COGS estimates</strong> are derived from [explain: publicly disclosed supplier revenue from target, analyst estimates, industry reports, proportional analysis of supplier revenue vs. target COGS]. Where exact figures are undisclosed, we estimate based on [methodology]. Figures marked with "~" are estimates.</p>
    <p style="margin-top:6px;"><strong>Concentration risk</strong> is assessed as HIGH (>15% of COGS or sole source), MED (5-15% of COGS or limited alternatives), LOW (<5% of COGS with multiple alternatives).</p>
  </div>
</div>
```

### Section 7: Tab Content — Customer Detail
```html
<div id="customers" class="tab-content">
  <h2>[TARGET] as Supplier &mdash; Key Customers</h2>
  <p style="color:var(--text-secondary); margin-bottom:16px;">Companies where [TARGET] is a significant supplier or vendor.</p>

  <!-- Customer cards with similar structure to supplier cards -->
  <div class="two-col">
    <!-- Customer company cards -->
  </div>
</div>
```

### Section 8: Tab Content — Tier 2 Suppliers
```html
<div id="tier2" class="tab-content">
  <h2>Tier 2 Supply Chain</h2>
  <p style="color:var(--text-secondary); margin-bottom:16px;">Key suppliers to [TARGET]'s major Tier 1 suppliers. These companies are two steps removed from [TARGET] in the supply chain.</p>

  <!-- For each Tier 1 supplier that has Tier 2 data -->
  <div id="tier2-[TICKER]" style="margin-bottom:32px;">
    <h3>[Tier 1 Supplier Name] ([TICKER]) &mdash; Their Key Suppliers</h3>
    <!-- Compact cards or table for each Tier 2 supplier -->
  </div>
</div>
```

### Section 9: Tab Content — Concentration Analysis
```html
<div id="concentration" class="tab-content">
  <h2>Supply Chain Concentration Analysis</h2>

  <div class="two-col">
    <div>
      <h3>Supplier Concentration</h3>
      <p style="font-size:13px; color:var(--text-secondary); margin-bottom:12px;">How dependent is [TARGET] on its top suppliers?</p>
      <!-- Bar chart or table showing % of COGS by supplier -->
      <!-- Flag any single supplier >20% as high risk -->
    </div>
    <div>
      <h3>Revenue Dependency</h3>
      <p style="font-size:13px; color:var(--text-secondary); margin-bottom:12px;">Which suppliers are most dependent on [TARGET]?</p>
      <!-- Bar chart or table showing [TARGET] as % of each supplier's revenue -->
      <!-- Flag any supplier where [TARGET] is >25% of revenue -->
    </div>
  </div>

  <!-- Risk Summary Box -->
  <div class="methodology-box" style="margin-top:24px;">
    <h4>Key Supply Chain Risks</h4>
    <ul style="padding-left:16px; margin-top:8px;">
      <li><strong>Single-source dependencies:</strong> [List any sole-source suppliers]</li>
      <li><strong>Geographic concentration:</strong> [Where are key suppliers located? Taiwan, China, etc.]</li>
      <li><strong>Revenue dependency:</strong> [Which suppliers would be most impacted if TARGET reduced orders?]</li>
    </ul>
  </div>
</div>
```

### Section 10: Footer
```html
  <div class="page-footer">
    Data sourced from <a href="https://daloopa.com">Daloopa</a>. All financial figures link to original source filings.
    Prepared [Date]. Supply chain relationships are based on public filings, analyst research, and industry reports. Not investment advice.
  </div>
</div><!-- end container -->
</body>
</html>
```

---

## CRITICAL RULES

### Citation Rules
- EVERY financial figure from Daloopa MUST be wrapped in `<a href="https://daloopa.com/src/{fundamental_id}">[value]</a>`
- Use the `id` field from `get_company_fundamentals` response as the `fundamental_id`
- Calculated metrics (margins, growth rates, % estimates derived from multiple sources) do NOT need citation links
- Document search results must cite: `<a href="https://marketplace.daloopa.com/document/{document_id}">[Filing]</a>`
- Never fabricate fundamental IDs — only use IDs returned from the API
- For estimated figures (% of COGS, % of revenue), always explain the methodology in the detail panel

### Financial Formatting
- Revenue/income in billions: `$XXB` or `$X.XB`
- Revenue/income in millions: `$X,XXX` (with comma separator)
- Margins: `XX.X%` (one decimal place)
- Percentages for supply chain: `~XX%` (use tilde for estimates)
- Use `&ndash;` for ranges, `&mdash;` for em-dashes, `&middot;` for separators

### Concentration Risk Classification
- **HIGH** (red): >15% of COGS, sole/single source, or geopolitical risk
- **MED** (amber): 5-15% of COGS, limited alternatives, or moderate switching costs
- **LOW** (green): <5% of COGS, multiple alternatives, easy to switch

### Supply Chain Data Quality
- Always distinguish between **disclosed** (from 10-K, investor reports) and **estimated** (from analyst research, proportional analysis)
- When estimating % of COGS, show your math: "TSMC Apple revenue ~$70B (per TSMC 10-K customer disclosure) / Apple TTM COGS ~$200B = ~35%"
- Use `~` prefix for all estimates
- Include source attribution for every data point in the detail panel
- If a figure cannot be reliably estimated, say "Not disclosed" rather than guessing

### Interactivity Rules
- Every company card must be clickable to expand
- Expanded cards show: full business description, relationship details, financial metrics, methodology notes
- "Explore Tier 2" button must navigate to the Tier 2 tab and scroll to the relevant section
- Tab switching must be smooth and not reload the page

---

## EXECUTION SEQUENCE

Follow this exact sequence:

1. **discover_companies** → get target company_id
2. **discover_company_series + get_company_fundamentals** → pull target company financials (revenue, COGS, margins)
3. **search_documents + WebSearch** → identify suppliers (run in parallel, multiple queries)
4. **discover_companies** → look up each identified supplier by ticker (batch)
5. **discover_company_series + get_company_fundamentals** → pull financials for each supplier (batch, parallel)
6. **search_documents + WebSearch** → determine revenue concentration for each supplier (parallel)
7. **WebSearch** → identify customers where target is a supplier
8. **discover_companies + get_company_fundamentals** → pull customer financials
9. **Repeat lighter version of 3-6** for Tier 2 (top 3-5 Tier 1 suppliers' suppliers)
10. **Synthesize** → organize all data into the framework
11. **Write HTML** → generate the complete self-contained HTML file
12. **Save & Open** → write to `~/Generated Stuff/[TICKER]-supply-chain.html` and open in browser

For steps 2-9, maximize parallelism — run independent searches and API calls concurrently.

After writing the file, always end with:
> Data sourced from Daloopa
