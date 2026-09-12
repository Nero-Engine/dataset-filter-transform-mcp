# Dataset Filter & Transform (Remote MCP Server)

**Clean, filter and reshape messy JSON rows in a single tool call.** Hand it a list of rows from a scraper, an API or a spreadsheet, tell it how to reshape them and which rows to keep, and it hands back clean rows plus an exact account of what was removed and why.

Built for AI agents. No install, no API key, no signup. Connect by URL and call it.

```
https://dataset-filter-transform.nerolabs.workers.dev/mcp
```

Free to use while in early access.

## What it does

One call runs a full pipeline, in this order:

1. **Transform** every row: rename, drop or keep fields, trim, change case, cast strings to numbers, compute new fields with arithmetic, extract with regex, split, replace, format dates, measure date gaps, pull the domain from an email or URL, hash, map values, parse JSON, and more (26 operations).
2. **Filter** out the rows you do not want: equals, contains, starts with, greater than, between, is empty, matches regex, in a list, array contains, date before or after, within the last N days, and more (25 operators), combined with AND or OR.
3. **Sort**, **de-duplicate** on any fields, and **offset / limit**.

Transforms run before filters, so you can create a field and filter on it in the same call.

It is honest about what it could not do. If a price like `"not a price"` cannot become a number, the row is not silently guessed at: the summary reports `cast:price: 1`.

## Tools

| Tool | What it does |
|---|---|
| `list_capabilities` | Lists every transform operation and filter operator, the pipeline order and the row limit. Processes no data. |
| `process_rows` | Runs the pipeline on the rows you pass and returns the kept rows plus a summary. |

## Connect

**Claude Code**

```bash
claude mcp add --transport http dataset-filter-transform https://dataset-filter-transform.nerolabs.workers.dev/mcp
```

**Claude Desktop / claude.ai:** Settings, Connectors, Add custom connector, paste the URL above.

**Cursor, Windsurf, VS Code and other MCP clients**

```json
{
  "mcpServers": {
    "dataset-filter-transform": {
      "url": "https://dataset-filter-transform.nerolabs.workers.dev/mcp"
    }
  }
}
```

## Example

Five messy lead rows go in:

```json
{
  "rows": [
    {"company": "  Acme Ltd  ", "e_mail": "Sales@ACME.co.uk", "price": "$1,234.50", "qty": "3", "country": "uk"},
    {"company": "Beta Systems", "e_mail": "hi@beta.io", "price": "49 USD", "qty": "1", "country": "UK"},
    {"company": "Gamma GmbH", "e_mail": "info@gamma.de", "price": "€900", "qty": "2", "country": "DE"},
    {"company": "Acme Ltd", "e_mail": "sales@acme.co.uk", "price": "$1,234.50", "qty": "3", "country": "UK"},
    {"company": "Delta Co", "e_mail": "", "price": "not a price", "qty": "5", "country": "UK"}
  ],
  "transforms": [
    {"op": "trim", "field": "company"},
    {"op": "rename", "from": "e_mail", "to": "email"},
    {"op": "lowercase", "field": "email"},
    {"op": "cast", "field": "price", "to": "number"},
    {"op": "cast", "field": "qty", "to": "number"},
    {"op": "compute", "field": "orderValue", "expression": "price * qty", "round": 2},
    {"op": "extractDomain", "field": "email", "into": "domain"}
  ],
  "filters": [
    {"field": "country", "operator": "equals", "value": "UK"},
    {"field": "orderValue", "operator": "greaterThan", "value": 100}
  ],
  "sortBy": [{"field": "orderValue", "direction": "desc"}],
  "distinctBy": ["domain"]
}
```

One clean row comes out, with a summary of exactly why the other four went:

```json
{
  "rows": [
    {"company": "Acme Ltd", "price": 1234.5, "qty": 3, "country": "uk", "email": "sales@acme.co.uk", "orderValue": 3703.5, "domain": "acme.co.uk"}
  ],
  "summary": {
    "inputRowCount": 5,
    "keptRowCount": 1,
    "excludedByFilter": 3,
    "duplicatesRemoved": 1,
    "transformIssues": {"cast:price": 1, "compute:orderValue": 1, "extractDomain:email": 1}
  }
}
```

Text matching is case-insensitive by default, and values like `"$1,234.50"`, `"49 USD"` and `"12%"` are read as numbers unless you turn that off.

## Limits

Up to **500 rows per call**. For bigger lists, split them across several calls. Anything larger returns a clear message rather than failing silently.

## Also available

The same engine runs on the Apify Store as [Dataset Filter & Transform](https://apify.com/nerolabs/dataset-filter-transform), which also reads Apify datasets, CSV and Excel files and Google Sheets, and exports CSV or Excel.

Built by **Nero Labs**.
