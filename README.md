# app-privacy-report-dashboard

# App Privacy Report Dashboard (NDJSON)

A **single‑file, client‑side dashboard** for exploring Apple’s *App Privacy Report* exports (`.ndjson`).
All parsing and analysis happens **locally in your browser** — no uploads, no servers, no tracking.

---

## What this dashboard does

This dashboard parses Apple’s exported **App Privacy Report NDJSON** and surfaces meaningful, human‑readable statistics about:

* 🌐 **Network activity** (domains, owners, apps)
* 🕸️ **Website network activity** (in‑app web browsing)
* 📡 **Sensor & data access** (Location, Camera, Microphone, etc.)
* 🧭 **Location access ranking** (most important privacy signal)

It is designed to be:

* **Fast** (chunked parsing, no blocking UI)
* **Defensive** (handles schema drift across iOS versions)
* **Auditable** (simple JS, no dependencies)

---

## Supported input

### File type

* Apple‑exported **App Privacy Report**
* File extension: `.ndjson` (newline‑delimited JSON)

Each line must be a standalone JSON object.

### Known Apple record types

The parser recognizes and processes the following:

| Record type      | How it’s detected               | Meaning                                          |
| ---------------- | ------------------------------- | ------------------------------------------------ |
| Network activity | `type === "networkActivity"`    | Domains contacted by apps or in‑app content      |
| TCC access       | `stream` contains `.stream.tcc` | Permission‑gated access (Contacts, Photos, etc.) |
| Access records   | `type === "access"`             | Data & sensor access (Location, Camera, Mic…)    |

Unknown records are ignored safely.

---

## Dashboard sections (what each table means)

### 1️⃣ Most contacted domains

**What it shows**
Domains ranked by **total network hits** across all apps.

**Derived from**

* `type: "networkActivity"`
* `domain`
* `hits`

**Columns**

* Domain
* Hits (sum)
* Owner (via Apple attribution)
* Top App (the app responsible for the most hits)

---

### 2️⃣ Most contacted domain owners

**What it shows**
Organizations ranked by total network activity.

**Derived from**

* `domainOwner`

Useful for spotting:

* analytics providers
* ad networks
* CDNs

---

### 3️⃣ Network activity by app (hits)

**What it shows**
Apps ranked by **total number of network hits**.

**Good for**

* identifying chatty apps
* spotting background network usage

---

### 4️⃣ App network activity (apps that contact the most domains)

**What it shows**
Apps ranked by **unique domains contacted**, then by total hits.

This is often more revealing than raw hit counts.

**Why it matters**

* 1 app contacting 5 domains ≠ 1 app contacting 200 domains

---

### 5️⃣ Website network activity (in‑app browsing)

**What it shows**
Websites visited **inside apps**, ranked by activity.

**How it’s inferred**

* Uses `context` field from `networkActivity`
* Treated as a website if it *looks like a domain* and differs from `domain`

**Columns**

* Website (context)
* Records (number of related network records)
* Total hits
* Top contacted domain

> ⚠️ Apple does not explicitly label “website network activity” in NDJSON. This is a **best‑effort heuristic** matching Apple’s UI.

---

### 6️⃣ TCC data & sensor access by app

**What it shows**
Apps ranked by **number of permission‑gated access events**.

**Derived from**

* `stream: com.apple.privacy.accounting.stream.tcc`

Examples:

* Contacts
* Photos
* Bluetooth

---

### 7️⃣ Top TCC services

**What it shows**
Which permissions are accessed most frequently.

Useful for understanding:

* what kinds of data are accessed overall
* which permissions dominate your device activity

---

### 8️⃣ Top access categories (type: "access")

**What it shows**
Counts of **Data & Sensor Access** categories.

**Derived from**

```json
{
  "type": "access",
  "category": "location",
  "accessor": { "identifier": "com.example.app" }
}
```

Common categories:

* location
* camera
* microphone
* motion

---

### 9️⃣ Sensor access: Location (⭐ most important)

**What it shows**
Apps ranked by how often they access **location**.

**Derived from**

* `type: "access"`
* `category: "location"`

**Why this matters**
Location access is:

* high‑risk
* often backgrounded
* frequently overused

This table makes it immediately obvious **which apps track location the most**.

---

### 🔟 Context table

**What it shows**
Raw `context` values from network activity, ranked by hits.

**Use case**

* debugging
* understanding how Apple is grouping in‑app activity

---

## Time window filtering

You can filter results by:

* Last 7 days
* Last 14 days
* Last 30 days
* All time (default)

Filtering uses:

* `timeStamp`
* `timestamp`
* `firstTimeStamp`

If no timestamp exists, the record is included by default.

---

## Parsing behavior & robustness

### Chunked parsing

* Parses files in chunks of **2,500 lines**
* Prevents browser freezes
* Shows live progress

### BOM / hidden character handling

Before parsing each line, the parser strips:

* UTF‑8 BOM (`\uFEFF`)
* NUL bytes (`\u0000`)

This fixes a common Apple export issue.

### Invalid / unknown lines

* Invalid JSON lines are skipped
* Unknown record types are ignored
* Counts are surfaced in the UI

No crashes, no partial renders.

---

## Privacy & security

* ✅ **100% local** parsing
* ❌ No network requests
* ❌ No analytics
* ❌ No storage

You can verify this by opening DevTools → Network tab.

---

## Browser compatibility

Tested on:

* Chrome / Chromium
* Edge
* Firefox
* Safari (macOS)

Requires:

* ES2020+
* File API

---

## Known limitations

* Website network activity is inferred heuristically
* Access durations are not computed (intervalBegin / intervalEnd pairing not yet implemented)
* Owner attribution depends on Apple’s `domainOwner` field

---

## Suggested future enhancements

* ⏱️ Location access **duration** (interval pairing)
* 🧠 Foreground vs background location detection
* 📊 Timeline charts
* 🔍 App‑level drill‑downs
* 📁 Export filtered views to CSV
