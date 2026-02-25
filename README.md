<div align="center">

<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:0f0c29,30:302b63,100:24243e&height=220&section=header&text=Asana%20%E2%87%84%20Google%20Sheets%20Engine&fontSize=50&fontColor=ffffff&fontAlignY=36&desc=Real-Time%20Bidirectional%20Sync%20%7C%20Automated%20Production%20Pipeline&descAlignY=56&descSize=16&animation=fadeIn" />

<br/>

[![Typing SVG](https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=700&size=20&duration=3000&pause=1000&color=A78BFA&center=true&vCenter=true&width=700&lines=⚡+Bidirectional+Sync+Engine;🤖+Zero+Human+Intervention;🧠+Smart+Diffing+%26+Mutation+Checks;🚀+Built+for+High-Velocity+Creative+Teams)](https://github.com/ar-rehman786)

<br/>

<p>
  <img src="https://img.shields.io/badge/Status-Production%20Live-22c55e?style=for-the-badge&logo=circle&logoColor=white" />
  &nbsp;
  <img src="https://img.shields.io/badge/Role-AI%20Automation%20Project-7c3aed?style=for-the-badge&logo=robot&logoColor=white" />
  &nbsp;
  <img src="https://img.shields.io/badge/Tech-n8n%20%7C%20JS%20%7C%20REST%20API-5D3EFF?style=for-the-badge&logo=n8n&logoColor=white" />
</p>

</div>

---

## 🧬 Overview

This repository showcases an enterprise-grade, **bidirectional synchronization engine** built in **n8n**. It acts as the central nervous system connecting a high-velocity creative ad production team's **Asana Workspace** with their **Google Sheets ("Creative Roadmap")**.

Instead of manual data entry and disjointed communication, this automation ensures that a change in one platform propagates to the other—flawlessly and autonomously.

**The Philosophy:** *If a human does it repeatedly — automate it.*

---

## 🎯 The Business Problem & ROI

### The Challenge
Managing hundreds of creative briefs across four dynamic pipeline stages (`Ideation` → `Filming` → `Editing` → `Advertising`) typically results in:
- ❌ **Data Silos:** Leadership lives in Google Sheets; Creatives live in Asana.
- ❌ **Sync Lag:** Someone has to manually update statuses, copy URLs, and assign editors.
- ❌ **Infinite Loops:** Poorly designed zaps/automations trigger each other in a never-ending loop, consuming API quotas.

### The Solution (This Pipeline)
- ✅ **Real-Time Webhooks:** Asana changes sync to Sheets instantly.
- ✅ **Smart Polling:** Sheet changes are picked up every 60 seconds and pushed to Asana. 
- ✅ **10-Point Smart Filter Engine:** Only payload diffs trigger a database mutation, saving 90% in server execution costs and eliminating loopbacks.
- ✅ **Zero Human Clicks:** Creates tasks, calculates deadlines (e.g., "Next Friday"), sets Enums/Custom Fields, and assigns Workspace Tags completely autonomously.

---

## 🚀 System Architecture

<div align="center">

> 🏭 **Dual-Engine Pipeline** · Triggers via **Webhooks & 1-Min Crons** · Processes **Briefs & Tasks** · **100% Autonomous**

</div>

```text
╔══════════════════════════════════════════════════════════════════════════╗
║                   BIDIRECTIONAL CREATIVE SYNC ENGINE                     ║
╠═══════════╦══════════════╦═══════════════════╦══════════════════════════╣
║ DIRECTION ║   TRIGGER    ║    PROCESSING     ║        DELIVER           ║
╠═══════════╬══════════════╬═══════════════════╬══════════════════════════╣
║ Asana →   ║ ⚡ Webhook   ║ 🧠 Extract GID    ║ 📊 Update Google Sheet   ║
║ Sheets    ║   (Instant)  ║ ⚖️ 10-Point Diff  ║    (Status, Assignees)   ║
╠═══════════╬══════════════╬═══════════════════╬══════════════════════════╣
║ Sheets →  ║ ⏰ 1 Min     ║ 🔀 Route: Create  ║ 📝 Set Custom Fields     ║
║ Asana     ║    Cron      ║    or Update      ║ 🏷️ Attach Tags           ║
╚═══════════╩══════════════╩═══════════════════╩══════════════════════════╝
```

### �️ Automation Visuals

> **HINT:** Drag and drop your workflow screenshots in the spaces below to showcase your pipelines on GitHub!

<details open>
<summary><strong>👉 Workflow 1: Asana → Google Sheets (The Webhook Engine)</strong></summary>

<br>

<!-- ============================================== -->
<!-- 💡 DRAG AND DROP YOUR "ASANA TO SHEETS" WORKFLOW IMAGE HERE 👇 -->
<!-- ============================================== -->



<!-- ============================================== -->

<br>

</details>

<details open>
<summary><strong>👉 Workflow 2: Google Sheets → Asana (The Polling Engine)</strong></summary>

<br>

<!-- ============================================== -->
<!-- 💡 DRAG AND DROP YOUR "SHEETS TO ASANA" WORKFLOW IMAGE HERE 👇 -->
<!-- ============================================== -->



<!-- ============================================== -->

<br>

</details>

---

## 🧠 Engineering & Logic Deep Dive

### 1️⃣ The "Smart Diffing" Filter (Asana → Sheets)
To prevent rate limits and infinite loops, I built a JavaScript diffing node that compares **10 distinct data points** between the live Asana payload and the existing Google Sheet row. 

**The filter ensures a write operation ONLY if:**
```javascript
✅  Status changed (Pipeline Stage moved)
✅  Tags changed (Author/Editor reassigned)
✅  Custom Fields updated (Priority, Format, Ad Concept)
✅  Drive or Doc URLs were added to the task notes
```
*If all match, the workflow halts gracefully, saving compute overhead.*

### 2️⃣ Dynamic Deadline Calculation (Sheets → Asana)
When a new row is added in Sheets, the workflow automatically generates an Asana task and calculates the deadline as **"The Upcoming Friday"**. 

```javascript
// Logic applied within the n8n Date node:
If Today = Mon-Thu --> Set Due Date to this week's Friday.
If Today = Fri-Sun --> Set Due Date to next week's Friday.
```

### 3️⃣ Complex Payload Parsing 
Asana Webhook payloads are deeply nested. A recursive JavaScript function was implemented to walk the object tree and extract the deepest `gid` representing the actual task event, filtering out noise.

### 4️⃣ REST API Overrides
Standard n8n nodes often fail to handle Enum-type custom fields in Asana perfectly. I engineered direct `HTTP Request` nodes using `PUT` methods to the Asana REST API to manually map and enforce custom field variables and workspace tags accurately.

---

## �️ Tech Stack & Tooling

<div align="center">

| Layer | Tools Used |
|:---|:---|
| 🔗 **Orchestration** | ![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white) |
| 📊 **Database** | ![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=for-the-badge&logo=google-sheets&logoColor=white) |
| 📋 **Project Mgmt** | ![Asana](https://img.shields.io/badge/Asana-F06A6A?style=for-the-badge&logo=asana&logoColor=white) ![Asana REST API](https://img.shields.io/badge/Asana_REST_API-1e3a5f?style=for-the-badge&logo=asana&logoColor=white) |
| ⚙️ **Logic / Parsing** | ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black) ![Regex](https://img.shields.io/badge/Regex-000000?style=for-the-badge) |

</div>

---

## � Deployment & Configuration

Want to deploy this system? Here is what you need:

1. **n8n Instance:** Self-hosted or Cloud.
2. **Authentication:** 
   - Asana OAuth2
   - Google Sheets OAuth2 (for reading/writing)
   - Google Sheets Trigger OAuth2 (for polling changes)
3. **Workspace Mapping:** Update the GIDs inside the HTTP Request nodes to match your specific Asana Workspace, Project, and Custom Enum Option IDs.
4. **Boot Sequence:** 
   - Enable the **Asana → Sheets** webhook workflow *first*.
   - Enable the **Sheets → Asana** polling workflow *second*.

---

<div align="center">

### 💡 Developed by [Abdul Rehman](https://github.com/ar-rehman786)
**AI Automation Engineer | Building Systems That Work While You Sleep**

[![Email](https://img.shields.io/badge/Email_Me-abdulrehmanhameed4321%40gmail.com-D14836?style=for-the-badge&logo=gmail&logoColor=white)](mailto:abdulrehmanhameed4321@gmail.com)
&nbsp;
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/ar-rehman786)
&nbsp;
[![Slora AI](https://img.shields.io/badge/🚀_Slora_AI-sloraai.com-5D3EFF?style=for-the-badge)](https://www.sloraai.com/)

*"The best engineer isn't the one who writes the most code — it's the one who removes the need for it."*

</div>

<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:24243e,50:302b63,100:0f0c29&height=120&section=footer&animation=fadeIn" />
