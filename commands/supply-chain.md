---
description: Create an interactive supply chain dashboard for any public company
argument-hint: Company ticker (e.g., "AAPL" or "TSLA" or "NVDA")
---

# Supply Chain Dashboard Generator

Generate an interactive supply chain map and financial dashboard as an HTML document.

**Request**: $ARGUMENTS

Follow the supply-chain skill workflow to produce a comprehensive interactive supply chain dashboard. The dashboard must:

1. **Identify the company**: Use `discover_companies` to find the company by ticker or name.
2. **Research supply chain**: Use Daloopa `search_documents` to find supplier mentions in 10-K, 10-Q filings. Simultaneously run web searches to identify key suppliers and customers.
3. **Map supplier relationships**: For each identified supplier, determine:
   - What they supply to the target company
   - Estimated % of target company's total COGS attributable to that supplier (with logic/citation)
   - What % of the supplier's revenue comes from the target company
4. **Map customer relationships**: Identify companies where the target is a major supplier.
5. **Pull financials**: For each company in the supply chain, pull key financial data from Daloopa MCP (revenue, margins, net income).
6. **Build interactive dashboard**: Generate a self-contained HTML file with:
   - Interactive supply chain network visualization
   - Click-to-expand company cards with financials, business description, products supplied
   - Navigation from 1st-order to 2nd-order suppliers
   - Clean, newspaper-style design (light mode, serif/sans-serif professional typefaces)
7. **Deliver**: Write the HTML file to `~/Generated Stuff/[TICKER]-supply-chain.html` and open it in the browser.

Every financial figure must include a Daloopa citation hyperlink. The dashboard should be information-dense and analytically rigorous.
