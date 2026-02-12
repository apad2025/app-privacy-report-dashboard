# App Privacy Report Dashboard (NDJSON)

A **single‑file, client‑side dashboard** for exploring Apple’s *App Privacy Report* exports (`.ndjson`).
All parsing and analysis happens **locally in your browser** — no uploads, no servers, no tracking.

# App Privacy Report Dashboard (NDJSON)

A **single-file, client-side dashboard** for exploring Apple’s *App Privacy Report* exports (`.ndjson`).
All parsing and analysis happens **locally in your browser** — no uploads, no servers, no tracking.

## What this dashboard does

This dashboard parses Apple’s exported **App Privacy Report NDJSON** and surfaces meaningful, human-readable statistics about:

* 🌐 **Network activity** (domains, owners, apps)
* 🕸️ **Website network activity** (in-app web browsing)
* 📡 **Sensor & data access** (Location, Camera, Microphone, etc.)
* 🧭 **Location access ranking** (most important privacy signal)

It is designed to be:

* **Fast** (chunked parsing, no blocking UI)
* **Defensive** (handles schema drift across iOS versions)
* **Auditable** (simple JS, no dependencies)

## Supported input

### File type

* Apple-exported **App Privacy Report**
* File extension: `.ndjson` (newline-delimited JSON)

Each line must be a standalone JSON object.

### Known Apple record types

The parser recognizes and processes the following:

| Record type      | How it’s detected               | Meaning                                          |
| ---------------- | ------------------------------- | ------------------------------------------------ |
| Network activity | `type === "networkActivity"`    | Domains contacted by apps or in-app content      |
| TCC access       | `stream` contains `.stream.tcc` | Permission-gated access (Contacts, Photos, etc.) |
| Access records   | `type === "access"`             | Data & sensor access (Location, Camera, Mic…)    |

Unknown records are ignored safely.

## Dashboard sections

### Most contacted domains

Domains ranked by **total network hits** across all apps.

Derived from:

* `type: "networkActivity"`
* `domain`
* `hits`

### Most contacted domain owners

Organizations ranked by total network activity.

Derived from:

* `domainOwner`

Useful for spotting analytics providers, ad networks, and CDNs.

### Network activity by app (hits)

Apps ranked by **total number of network hits**.

Useful for identifying chatty apps or background network usage.

### App network activity (unique domains)

Apps ranked by **unique domains contacted**, then by total hits.

This is often more revealing than raw hit counts.

### Website network activity (in-app browsing)

Websites visited **inside apps**, inferred from the `context` field.

The dashboard treats a context as a website if it looks like a domain and differs from the contacted domain.

Apple does not explicitly label website activity in NDJSON, so this is a best-effort heuristic that closely matches the Settings UI.

### TCC data & sensor access by app

Apps ranked by **permission-gated access events**.

Derived from:

* `stream: com.apple.privacy.accounting.stream.tcc`

Examples include Contacts, Photos, Bluetooth, and more.

### Top TCC services

Permission types ranked by access frequency.

Helps explain *what kinds of data* are accessed most often overall.

### Access categories (type: "access")

Counts of Data & Sensor Access categories.

Derived from records such as:

```json
{"type":"access","category":"location","accessor":{"identifier":"com.example.app"}}
```

Common categories include location, camera, microphone, and motion.

### Sensor access: Location

Apps ranked by how often they access **location**.

Derived from:

* `type: "access"`
* `category: "location"`

This is the most important privacy signal surfaced by the dashboard.

### Context table

Raw `context` values from network activity, ranked by hits.

Primarily useful for debugging and understanding Apple’s attribution model.

## Time window filtering

Results can be filtered by:

* Last 7 days
* Last 14 days
* Last 30 days
* All time (default)

Filtering uses the first valid timestamp among:

* `timeStamp`
* `timestamp`
* `firstTimeStamp`

If a record has no timestamp, it is included by default.

## Parsing behavior

* Files are parsed in **2,500-line chunks** to keep the UI responsive
* UTF-8 BOM (`\uFEFF`) and NUL characters (`\u0000`) are stripped before parsing
* Invalid JSON lines are skipped safely
* Unknown record types are ignored without breaking the dashboard

## Privacy

* Parsing happens entirely **in your browser**
* No network requests are made
* No data is stored or transmitted

You can verify this via your browser’s Network tab.

## Browser compatibility

Tested on:

* Chrome / Chromium
* Edge
* Firefox
* Safari (macOS)

Requires modern JavaScript (ES2020+) and the File API.

## Limitations

* Website network activity is inferred heuristically
* Access durations are not computed (interval pairing not implemented)
* Domain owner attribution depends on Apple’s metadata

## Future ideas

* Location access duration (intervalBegin / intervalEnd pairing)
* Foreground vs background location detection
* Timeline visualizations
* App-level drill-down views
* CSV export of filtered results

## License

MIT: use freely, attribution appreciated.
