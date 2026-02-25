<div align="center">

<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:0f2027,40:203a43,100:2c5364&height=200&section=header&text=Asana%20%E2%87%84%20Google%20Sheets%20Sync&fontSize=40&fontColor=ffffff&fontAlignY=38&desc=Real-time%20bidirectional%20sync%20for%20creative%20ad%20production%20pipelines&descAlignY=58&descSize=14&animation=twinkling" />

</div>

<div align="center">

[![Status](https://img.shields.io/badge/Status-Active-22c55e?style=for-the-badge&logo=circle&logoColor=white)](#)
[![Platform](https://img.shields.io/badge/n8n-Workflow-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](#)
[![Integration](https://img.shields.io/badge/Asana-Integration-F06A6A?style=for-the-badge&logo=asana&logoColor=white)](#)
[![Integration](https://img.shields.io/badge/Google_Sheets-Bidirectional-34A853?style=for-the-badge&logo=google-sheets&logoColor=white)](#)
[![Sync Time](https://img.shields.io/badge/Sync-Real_Time-3b82f6?style=for-the-badge&logo=clockify&logoColor=white)](#)

</div>

---

## 📖 Overview

This repository contains two robust **n8n workflows** that maintain perfect synchronization between an **Asana project board** and a **Google Sheet ("CREATIVE ROADMAP")**. 

Designed for high-velocity creative ad production teams, this system eliminates manual data entry and ensures that all stakeholders—whether they prefer spreadsheets or Kanban boards—always have the most up-to-date information across four pipeline stages: `Ideation` → `Filming` → `Editing` → `Advertising`.

---

## 📸 Workflow Previews

> **HINT:** Drag and drop your workflow screenshots into the spaces below!

<details open>
<summary><strong>👉 Workflow 1: Asana → Google Sheets</strong> (Click to collapse)</summary>

<br>

<!-- ============================================== -->
<!-- DRAG AND DROP YOUR WORKFLOW 1 IMAGE HERE 👇 -->
<!-- ============================================== -->



<!-- ============================================== -->

<br>

</details>

<details open>
<summary><strong>👉 Workflow 2: Google Sheets → Asana</strong> (Click to collapse)</summary>

<br>

<!-- ============================================== -->
<!-- DRAG AND DROP YOUR WORKFLOW 2 IMAGE HERE 👇 -->
<!-- ============================================== -->



<!-- ============================================== -->

<br>

</details>

---

## ✨ Key Features

- **🔄 Bidirectional Sync**: Changes in Asana reflect in Google Sheets instantly via webhooks. Changes in Sheets create or update Asana tasks within a 1-minute polling cycle.
- **🧠 Smart Diffing Engine**: Prevents infinite sync loops. Changes are only written if a 10-field diff check detects actual modifications.
- **🏷️ Dynamic Custom Fields & Tags**: Automatically maps and updates Asana custom fields via the REST API and attaches relevant tags (e.g., Author, Editor).
- **📅 Automated Deadlines**: Automatically calculates "next Friday" deadlines for new briefs created from the spreadsheet.
- **🔗 Relational Mapping**: Uses Asana Task GIDs as foreign keys within the Google Sheet to permanently link rows to tasks.

---

## 🏗️ Architecture

The integration consists of two distinct data flows working in harmony:

### 1️⃣ Asana → Google Sheets (Real-Time)
*Triggered instantly when a task is updated in Asana.*

1. **Webhook Listeners**: 4 separate triggers corresponding to pipeline stages listen for changes.
2. **GID Extraction**: Custom JavaScript recursively extracts the exact Task GID from the webhook payload.
3. **Data Fetching**: Pulls full task details (tags, custom fields, status) from the Asana API.
4. **Diff Check**: Compares 10 specific fields against the current Google Sheet row.
5. **Update**: If differences exist, updates the Sheet row silently.

### 2️⃣ Google Sheets → Asana (Every 1 Minute)
*Polls the CREATIVE ROADMAP for any row changes.*

1. **Watch Rows**: Detects changes in the range `A1:Z1000`.
2. **Routing**: Checks if the row has an `Asana Task GID`.
    - **No GID (Create)**: Computes due date, creates a new task in `Ideation`, updates custom fields/tags, and writes the generated GID back to the Sheet.
    - **Has GID (Update)**: Updates existing task notes, custom fields, and tags to match the Sheet.

---

## 📊 Data Schema & Mapping

### Custom Fields Mapping
Maps directly to Asana Workspace `1210018926299426`.

| Google Sheet Column | Asana Equivalent | Data Type |
| :--- | :--- | :--- |
| `Brief #` | Task Name | Text |
| `Status` | Section Name | Enum |
| `Type` | Custom Field `[0]` | Enum |
| `# of IT` | Custom Field `[1]` | Enum |
| `Format` | Custom Field `[2]` | Enum |
| `LP` | Custom Field `[3]` | Text |
| `Ad Concept` | Custom Field `[4]` | Text |
| `Launch Priority` | Custom Field `[5]` | Enum (1-5) |
| `Author` | Task Tag `[0]` | Tag |
| `Editor` | Task Tag `[1]` | Tag |
| `Brief URL` | Task Notes (Line 1) | URL |
| `DRIVE_URL` | Task Notes (Line 2) | URL |

---

## 🚀 Setup & Installation

### Prerequisites
- Self-hosted or Cloud **n8n** instance.
- **Asana** OAuth2 Credentials.
- **Google Sheets** and **Google Sheets Trigger** OAuth2 Credentials.

### Quick Start
1. **Import Workflows**: Import the JSON files for both workflows into n8n.
2. **Authenticate**: Connect your Asana and Google workspace accounts in the n8n credentials panel.
3. **Configure Targets**:
   - Update the Spreadsheet ID in all Google Sheets nodes to: `1fP2DpH7dZ9QjyS1HcIdRiGZ4an1JPwy7GHLu_tFa1PA`
   - Verify Asana Project GIDs in the webhook triggers (`Ideation`, `Editing`, `Filming`, `Advertising`).
4. **Deploy**:
   - ⚠️ **Activate Asana → Sheets first** to ensure webhooks are listening.
   - ⚠️ **Activate Sheets → Asana second** to begin 1-minute polling.

> **Security Note:** The hardcoded Bearer token in the HTTP nodes for Asana REST API calls should be moved to a securely stored n8n credential before production deployment.

---

<div align="center">
<img width="100%" src="https://capsule-render.vercel.app/api?type=waving&color=0:2c5364,50:203a43,100:0f2027&height=100&section=footer&animation=twinkling" />
</div>
