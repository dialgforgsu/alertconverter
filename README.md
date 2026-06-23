# Alert → Redgate Monitor Converter

A single-page web tool that converts alert/alarm exports from other SQL Server
monitoring products into **Redgate Monitor**–ready output.

Everything runs **in your browser** — no data is uploaded to any server.

## Supported sources

| Source tool | Accepted exports |
|---|---|
| **SolarWinds SQL Sentry** | Alerts log CSV, `.condition` files, ZIP |
| **IDERA SQL Diagnostic Manager** | CSV |
| **SolarWinds DPA** (Database Performance Analyzer) | CSV, native JSON (`Alerts_*.json`, `Rules_*.json`, `CustomProperties_*.json`) |
| **Quest Spotlight on SQL Server** | Alarm history CSV (Enterprise or Cloud) |

Each source alert/alarm is matched to its nearest Redgate Monitor equivalent and
labelled by match quality: **direct**, **partial**, **custom metric required**,
**no equivalent**, or **retire** (made redundant by Redgate Monitor's model).

## Output formats

- **PowerShell module (`.psm1`)** — signed-style module with `-WhatIf`, transcript
  logging, and a rollback log; pushes alerts as timeline annotations.
- **Webhook JSON** — for Redgate Monitor's webhook/notification integrations.
- **Annotation SQL** — T-SQL to record the alerts directly.

The tool also produces threshold-conversion guidance (e.g. days→hours,
minutes→seconds, % memory→MB available) and ready-to-paste T-SQL for any alert
that needs a custom metric.

## Usage

No build step or dependencies to install. Either:

1. Open `index.html` in any modern browser, **or**
2. Serve the folder statically (e.g. `python -m http.server`) and visit it.

Then:

1. **Pick the source tool** and drop in your export (CSV / ZIP / `.condition` /
   JSON), or paste raw CSV.
2. **Map columns** — auto-detected where names align; override as needed.
3. **Preview** the mapped alerts and uncheck any to exclude.
4. **Generate outputs** in your chosen format and copy or download.

To run the generated PowerShell, download the `RedgateSQM.psm1` module and an
auth token from **Configuration → PowerShell API** in your Redgate Monitor
portal, place the module beside the generated `.ps1`, and run it from any
machine that can reach the server.

## Project files

| File | Purpose |
|---|---|
| `index.html` | The entire application (UI, mapping tables, generators). |
| `ss-rg-mapping-manifest.json` | SQL Sentry → Redgate Monitor mapping reference. |
| `idera-rg-mapping-manifest.json` | IDERA → Redgate Monitor mapping reference. |
| `dpa-rg-mapping-manifest.json` | DPA → Redgate Monitor mapping reference. |
| `quest-rg-mapping-manifest.json` | Quest Spotlight → Redgate Monitor mapping reference. |
| `advisory-conditions/` | SQL Sentry advisory conditions with T-SQL equivalents. |

## Notes

- Mappings target **Redgate Monitor 14**. Several alerts that previously required
  a custom metric are now native built-ins and map directly — including
  *Version store usage*, *Virtual log file count*, and the dedicated
  *Differential backup overdue* / *Log backup overdue* alerts.
- Not affiliated with SolarWinds, Quest, or Redgate. Product names belong to
  their respective owners.

## References

- [Redgate Monitor docs](https://documentation.red-gate.com/monitor14)
- [SQL Sentry docs](https://documentation.solarwinds.com/en/success_center/sqlsentry)
- [Quest Spotlight docs](https://help.spotlightessentials.com/sqlserver_alarms)
