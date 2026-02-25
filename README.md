<div align="center">

<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:0f2027,40:203a43,100:2c5364&height=200&section=header&text=Asana%20%E2%87%84%20Google%20Sheets%20Sync&fontSize=40&fontColor=ffffff&fontAlignY=38&desc=Real-time%20bidirectional%20sync%20for%20creative%20ad%20production%20pipelines&descAlignY=58&descSize=14&animation=twinkling" />

<br/>

<img src="https://img.shields.io/badge/Status-Live%20%26%20Active-22c55e?style=for-the-badge&logo=circle&logoColor=white" />
&nbsp;
<img src="https://img.shields.io/badge/n8n-Workflow-EA4B71?style=for-the-badge&logo=n8n&logoColor=white" />
&nbsp;
<img src="https://img.shields.io/badge/Asana-Webhook%20Trigger-F06A6A?style=for-the-badge&logo=asana&logoColor=white" />
&nbsp;
<img src="https://img.shields.io/badge/Google%20Sheets-Bidirectional%20Sync-34A853?style=for-the-badge&logo=google-sheets&logoColor=white" />
&nbsp;
<img src="https://img.shields.io/badge/Sync-Every%201%20Minute-3b82f6?style=for-the-badge&logo=clockify&logoColor=white" />

</div>

---

## Overview

Two n8n workflows that keep an Asana project board and a **Google Sheet ("CREATIVE ROADMAP")** in perfect sync — automatically, in both directions. Changes in Asana reflect in the sheet within seconds. Changes in the sheet create or update Asana tasks within a minute — without human intervention.

Built for a creative ad production team managing briefs across four pipeline stages: **Ideation → Filming → Editing → Advertising**.

---

## How the Two Workflows Fit Together

```
┌─────────────────────────────────────────────────────────────────┐
│                      CREATIVE ROADMAP                           │
│                       Google Sheet                              │
└──────────────┬──────────────────────────┬───────────────────────┘
               │  Excel → Asana           │  Asana → Excel
               │  (row change detected)   │  (task event webhook)
               ▼                          ▲
┌─────────────────────────────────────────────────────────────────┐
│   Ideation  │  Filming  │  Editing  │  Advertising   (Asana)   │
└─────────────────────────────────────────────────────────────────┘
```

| Workflow | Direction | Trigger | Primary Action |
|:---|:---|:---|:---|
| **Asana → Excel** | Asana → Sheet | Webhook (instant) | Update existing row when task fields change |
| **Excel → Asana** | Sheet → Asana | Poll every 1 min | Create new task or update existing one |

---

## 🖼️ Workflow Screenshots

<details>
<summary><strong>📊 Workflow 1 — Asana → Excel</strong> &nbsp;(click to expand)</summary>

<br/>

> 4 Asana triggers fan in → Code → Get a task → Read sheet row → Diff check → Update row

<img width="100%" src="pdf_screenshots/page2_img1.png" alt="Asana → Excel workflow canvas showing 4 triggers (Advertising, Filming, Editing, Ideation) flowing into Code node, Get a task, Get rows in sheet, Code1, IF diff gate, and Update row in sheet" />

</details>

<details>
<summary><strong>📋 Workflow 2 — Excel → Asana</strong> &nbsp;(click to expand)</summary>

<br/>

> Watch Rows → IF (create vs update) → variables → Due date → Create/Update task → Custom fields → Tags → Write GID back

<img width="100%" src="pdf_screenshots/page1_img1.jpeg" alt="Excel → Asana workflow canvas showing Watch Rows trigger, IF gate splitting into create path (Add variables, Due date1, Create a task1, custom fields, tags, Update row in sheet1) and update path (Add variables when update, Update a task, custom fields, tags)" />

</details>

---

## Workflow 1 — Asana → Excel

> *"Any time a task changes in Asana, the sheet row is updated automatically."*

### Architecture

```mermaid
flowchart LR
    subgraph TRIGGERS ["⚡ Asana Triggers (Webhook)"]
        T1["📋 Advertising"]
        T2["🎬 Filming"]
        T3["✂️ Editing"]
        T4["💡 Ideation"]
    end

    T1 & T2 & T3 & T4 --> CODE

    CODE["🔧 Code\nExtract last GID\nfrom webhook payload"]

    CODE --> TASK["📥 Get a task\nFull task details\nfrom Asana API"]

    TASK --> SHEET_READ["📊 Get row(s) in sheet\nLook up row\nby Asana Task GID"]

    SHEET_READ --> CODE1["🔧 Code1\nExtract Brief URL\n& Drive URL from notes"]

    CODE1 --> IF{"🔀 IF\n10-field diff check\nAnything changed?"}

    IF -- "✅ Yes → write" --> SHEET_WRITE["📊 Update row in sheet\nSync 11 fields"]
    IF -- "❌ No change" --> STOP(["⏭ Skip"])

    style T1 fill:#F06A6A,color:#fff,stroke:#d45a5a
    style T2 fill:#F06A6A,color:#fff,stroke:#d45a5a
    style T3 fill:#F06A6A,color:#fff,stroke:#d45a5a
    style T4 fill:#F06A6A,color:#fff,stroke:#d45a5a
    style CODE fill:#1e3a5f,color:#fff,stroke:#2c5364
    style TASK fill:#1e3a5f,color:#fff,stroke:#2c5364
    style SHEET_READ fill:#34A853,color:#fff,stroke:#2d8f47
    style CODE1 fill:#1e3a5f,color:#fff,stroke:#2c5364
    style IF fill:#f59e0b,color:#fff,stroke:#d97706
    style SHEET_WRITE fill:#34A853,color:#fff,stroke:#2d8f47
    style STOP fill:#444,color:#fff,stroke:#333
```

### Node-by-Node

#### 1 · Asana Triggers (×4)

Four separate webhook listeners — one per Asana project section. All feed into the same downstream pipeline.

| Trigger Node | Asana Project GID | Stage |
|:---|:---|:---|
| **Advertising** | `1211025036975704` | Final stage — ready to launch |
| **Filming** | `1210018922107046` | Content capture in progress |
| **Editing** | `1211025036975701` | Post-production |
| **Ideation** | `1211025036975698` | Brief writing & concept |

#### 2 · Code — Extract GID

The webhook payload is a nested object. This JavaScript recursively walks it to find the **deepest `gid`** value — the actual task ID that changed.

```js
function getLastGid(o) {
  let lastGid = null;
  for (const key of Object.keys(o)) {
    const value = o[key];
    if (value && typeof value === 'object' && !Array.isArray(value)) {
      lastGid = getLastGid(value); // recurse
    }
  }
  if (o.gid) lastGid = o.gid;
  return lastGid;
}
```

#### 3 · Get a task

Fetches full task detail from the Asana API using the extracted GID. Returns tags (Author, Editor), custom fields, section name (Status), and notes.

#### 4 · Get row(s) in sheet

Looks up the corresponding row in **CREATIVE ROADMAP** sheet, matched on `Asana Task GID` column.

#### 5 · Code1 — Extract URLs from Notes

Task notes store two URLs on separate lines. This regex pulls them out:

```
notes format:
1. https://docs.google.com/... (Brief URL)
2. https://drive.google.com/... (Drive URL)
```

#### 6 · IF — Smart Diff Gate (10 conditions, OR logic)

Only writes to the sheet if **at least one field has changed**. Compares:

| # | Sheet Column | Asana Source |
|:---|:---|:---|
| 1 | `Brief #` | `task.name` |
| 2 | `Editor` | `task.tags[1].name` |
| 3 | `Status` | `task.memberships[0].section.name` |
| 4 | `# of IT` | `custom_fields[1].display_value` |
| 5 | `Type` | `custom_fields[0].display_value` |
| 6 | `LP` | `custom_fields[3].display_value` |
| 7 | `Ad Concept` | `custom_fields[4].display_value` |
| 8 | `Launch Priority` | `custom_fields[5].display_value` (numeric) |
| 9 | `Brief URL` | First URL extracted from notes |
| 10 | `DRIVE_URL` | Second URL extracted from notes |

> If nothing changed (e.g. Asana fired an unrelated event), the row is silently skipped.

#### 7 · Update row in sheet

Writes all 11 synced fields to the matched row: `Status`, `Brief #`, `Author`, `Editor`, `# of IT`, `LP`, `Ad Concept`, `Launch Priority`, `Type`, `Format`, `Asana Task GID`.

---

## Workflow 2 — Excel → Asana

> *"New rows in the sheet become Asana tasks. Edited rows update existing tasks. Custom fields and tags are synced precisely."*

### Architecture

```mermaid
flowchart TD
    WATCH["📊 Watch Rows\nGoogle Sheets Trigger\nPoll every 1 min\nRange A1:Z1000"] --> IF_GID

    IF_GID{"🔀 IF\nAsana Task GID\nempty?"} -- "✅ Empty → CREATE" --> VARS_NEW
    IF_GID -- "❌ Has GID → UPDATE" --> VARS_UPDATE

    subgraph CREATE_PATH ["🆕 Create Task Path"]
        VARS_NEW["📝 Add variables\nExtract all brief fields"] --> DUE
        DUE["📅 Due date1\nCalculate next Friday"] --> CREATE
        CREATE["➕ Create a task1\nNew Asana task\nin Ideation project"] --> CF1
        CF1["🌐 Get all custom fields1\nFetch field GIDs"] --> MCF1
        MCF1["🔧 Matching custom fields1\nMap values → enum GIDs"] --> UCF1
        UCF1["📡 Update Custom Fields1\nPUT to Asana API"] --> TAGS1
        TAGS1["🌐 Get all tags1\nFetch workspace tags"] --> RT1
        RT1["🔧 Return matching tags\nFind Author + Editor tags"] --> ATT1
        ATT1["🏷 Add a task tag1\nAttach both tags"] --> URS1
        URS1["📊 Update row in sheet1\nWrite GID back to sheet"]
    end

    subgraph UPDATE_PATH ["✏️ Update Task Path"]
        VARS_UPDATE["📝 Add variables when update\nExtract all brief fields"] --> UPDATE
        UPDATE["✏️ Update a task\nSync notes with URLs"] --> CF2
        CF2["🌐 Get all custom fields\nFetch field GIDs"] --> MCF2
        MCF2["🔧 Matching custom fields\nMap values → enum GIDs"] --> UCF2
        UCF2["📡 Update Custom Fields\nPUT to Asana API"] --> TAGS2
        TAGS2["🌐 Get all tags\nFetch workspace tags"] --> RT2
        RT2["🔧 Return matching tags1\nFind Author + Editor tags"] --> ATT2
        ATT2["🏷 Add a task tag\nAttach both tags"]
    end

    style WATCH fill:#34A853,color:#fff,stroke:#2d8f47
    style IF_GID fill:#f59e0b,color:#fff,stroke:#d97706
    style VARS_NEW fill:#1e3a5f,color:#fff,stroke:#2c5364
    style DUE fill:#1e3a5f,color:#fff,stroke:#2c5364
    style CREATE fill:#F06A6A,color:#fff,stroke:#d45a5a
    style CF1 fill:#3b82f6,color:#fff,stroke:#2563eb
    style MCF1 fill:#1e3a5f,color:#fff,stroke:#2c5364
    style UCF1 fill:#3b82f6,color:#fff,stroke:#2563eb
    style TAGS1 fill:#3b82f6,color:#fff,stroke:#2563eb
    style RT1 fill:#1e3a5f,color:#fff,stroke:#2c5364
    style ATT1 fill:#F06A6A,color:#fff,stroke:#d45a5a
    style URS1 fill:#34A853,color:#fff,stroke:#2d8f47
    style VARS_UPDATE fill:#1e3a5f,color:#fff,stroke:#2c5364
    style UPDATE fill:#F06A6A,color:#fff,stroke:#d45a5a
    style CF2 fill:#3b82f6,color:#fff,stroke:#2563eb
    style MCF2 fill:#1e3a5f,color:#fff,stroke:#2c5364
    style UCF2 fill:#3b82f6,color:#fff,stroke:#2563eb
    style TAGS2 fill:#3b82f6,color:#fff,stroke:#2563eb
    style RT2 fill:#1e3a5f,color:#fff,stroke:#2c5364
    style ATT2 fill:#F06A6A,color:#fff,stroke:#d45a5a
```

### Node-by-Node

#### 1 · Watch Rows

Polls the **CREATIVE ROADMAP** sheet every minute for any row updates within `A1:Z1000`. Fires once per changed row.

#### 2 · IF — Create vs. Update

| Condition | Branch |
|:---|:---|
| `Asana Task GID` is **empty** | → Create new task |
| `Asana Task GID` is **present** | → Update existing task |

#### Create Path

**`Due date1`** — Calculates the deadline as **next Friday**:

```
Monday–Thursday  → this coming Friday
Friday           → the following Friday (+7)
Saturday         → next Friday (+6)
Sunday           → next Friday (+5)
```

**`Create a task1`** — Creates the task in the **Ideation** project with:
- Name = `Brief #`
- Notes = URL 1 + URL 2 from sheet
- Due date = computed Friday

**Custom Fields Pipeline (Create)**
```
Get all custom fields1  →  Matching custom fields1  →  Update Custom Fields1
(fetch enum GIDs)           (match text → GID)          (PUT to Asana API)
```
Maps 6 custom fields: `Type`, `# of IT (Variation)`, `Format`, `LP`, `Ad Concept`, `Launch Priority`

**Tags Pipeline (Create)**
```
Get all tags1  →  Return matching tags  →  Add a task tag1
(all tags)         (filter Author + Editor)   (attach both)
```

**`Update row in sheet1`** — Writes the new `Asana Task GID` back to the sheet, permanently linking the row ↔ task.

#### Update Path

**`Update a task`** — Updates task notes with current `Brief URL` and `DRIVE_URL` from sheet.

Then runs the same **custom fields** and **tags** pipeline as the Create path, keeping Asana in sync with any edits made directly in the sheet.

---

## Data Schema

### Google Sheet Columns (CREATIVE ROADMAP)

| Column | Source | Notes |
|:---|:---|:---|
| `Brief #` | Task name | e.g. `B5/AYB14` |
| `Author` | Tag 0 on task | Person who wrote the brief |
| `Editor` | Tag 1 on task | Person editing the video |
| `Status` | Section name | Pipeline stage |
| `# of IT` | `custom_fields[1]` | Iteration / variation count |
| `Type` | `custom_fields[0]` | Content type |
| `Format` | `custom_fields[2]` | e.g. `RV` (Reels Video) |
| `LP` | `custom_fields[3]` | Landing page |
| `Ad Concept` | `custom_fields[4]` | Creative concept |
| `Launch Priority` | `custom_fields[5]` | Number (1–5) |
| `Brief URL` | Task notes line 1 | Google Doc link |
| `DRIVE_URL` | Task notes line 2 | Google Drive folder |
| `Asana Task GID` | Created by workflow | Foreign key linking row ↔ task |
| `UUID` | Sheet-generated | Used to match row on creation |

### Custom Field ID Mapping (Asana workspace `1210018926299426`)

| Index | Field Name | Type |
|:---|:---|:---|
| `[0]` | Type | Enum |
| `[1]` | Variation (# of IT) | Enum |
| `[2]` | Format | Enum |
| `[3]` | Landing Page (LP) | Text |
| `[4]` | Ad Concepts | Text |
| `[5]` | Launch Priority | Enum |

---

## Sync Sequence — End to End

```mermaid
sequenceDiagram
    participant S as 📊 Google Sheet
    participant W2 as ⚡ Excel→Asana
    participant A as 🟧 Asana
    participant W1 as ⚡ Asana→Excel

    Note over S,W1: New brief added to sheet (no GID)
    S->>W2: Row change detected (1-min poll)
    W2->>A: Create task in Ideation
    W2->>A: Set custom fields (PUT)
    W2->>A: Attach Author + Editor tags
    W2->>S: Write Asana Task GID back to row

    Note over S,W1: Task moved to Editing in Asana
    A->>W1: Webhook fires (section change)
    W1->>A: Fetch full task details
    W1->>S: Read current row
    W1->>W1: Diff — Status changed ✅
    W1->>S: Update Status in sheet

    Note over S,W1: Editor name changed in sheet
    S->>W2: Row change detected
    W2->>A: Update task notes
    W2->>A: Update custom fields (PUT)
    W2->>A: Re-attach tags (new editor)
```

---

## Tech Stack

<div align="center">

| Tool | Role |
|:---|:---|
| ![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white) | Workflow orchestration |
| ![Asana](https://img.shields.io/badge/Asana-F06A6A?style=for-the-badge&logo=asana&logoColor=white) | Project management + webhook triggers |
| ![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=for-the-badge&logo=google-sheets&logoColor=white) | Creative roadmap data store |
| ![Asana REST API](https://img.shields.io/badge/Asana_REST_API-1e3a5f?style=for-the-badge&logo=asana&logoColor=white) | Custom field + tag updates (direct PUT) |
| ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black) | GID extraction, URL parsing, enum matching |

</div>

---

## Setup Guide

### Prerequisites

- [ ] n8n instance (self-hosted or cloud)
- [ ] Asana OAuth2 credentials configured in n8n
- [ ] Google Sheets OAuth2 credentials configured in n8n  
- [ ] Google Sheets Trigger OAuth2 credentials configured in n8n
- [ ] A copy of the CREATIVE RESEARCH [L-CART] spreadsheet

### Credentials Required

| Credential | Used by |
|:---|:---|
| `Asana account` (OAuth2) | Asana triggers, task read/write, tag operations |
| `Google Sheets account` (OAuth2) | Sheet read/update operations |
| `Google Sheets Trigger account` (OAuth2) | Watch Rows polling trigger |

### Configuration Steps

1. **Import both workflow JSONs** into n8n
2. **Connect credentials** — Asana OAuth2, Google Sheets OAuth2, Sheets Trigger OAuth2
3. **Update Sheet ID** in all Google Sheets nodes: `1fP2DpH7dZ9QjyS1HcIdRiGZ4an1JPwy7GHLu_tFa1PA`
4. **Verify Asana project GIDs** match your workspace:

   | Project | GID |
   |:---|:---|
   | Ideation | `1211025036975698` |
   | Editing | `1211025036975701` |
   | Filming | `1210018922107046` |
   | Advertising | `1211025036975704` |
   | Workspace | `1210018926299426` |

5. **Activate Asana → Excel first** — so webhook listeners are live before any tasks change
6. **Activate Excel → Asana** — polling begins immediately

> ⚠️ The Asana REST API Bearer token hardcoded in HTTP nodes should be replaced with a securely stored credential in production.

---

## Key Design Decisions

| Decision | Rationale |
|:---|:---|
| **Diff check before writing** | Prevents infinite sync loops — Asana events would re-trigger the sheet, which would re-trigger Asana |
| **GID as foreign key** | Single source of truth linking a sheet row to its Asana task permanently |
| **UUID for creation matching** | Allows writing the new GID back to the exact row that triggered the creation |
| **Direct Asana REST API for custom fields** | n8n's Asana node doesn't support enum custom field updates — HTTP PUT is necessary |
| **Next-Friday due date logic** | Aligns with the team's weekly sprint cycle |
| **Separate custom field + tag pipelines** | Field GIDs must be fetched fresh each run — workspace enum option IDs can't be hardcoded reliably |

---

<div align="center">

<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:2c5364,50:203a43,100:0f2027&height=100&section=footer&animation=twinkling" />

</div>
