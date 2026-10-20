---
name: solopreneur-operations
description: Use when architecting an AI-native one-person company.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [Solopreneur, One-Person-Company, Operating-Model, Business-Automation, Virtual-Workforce, Open-Source]
    related_skills: [obsidian, weekly-review-planning, email-inbox-triage, competitor-news-monitor, document-to-action-items]
---

# Solopreneur Operations & AI Workforce Architecture

Use this skill to design, organize, automate, and operate an AI-native One-Person Company (OPC). It guides structuring a founder-led business where AI agents and open-source tools execute operations while the founder retains strategic control and relationship ownership.

## Core Principle: Business First, Capabilities Second, Tools Third

Do not begin by listing AI coding agents or generic SaaS tools. Always map the business model and revenue-critical functions first, determine required capabilities second, and select matching open-source/local-first tools third.

---

## Operating Procedure

### 1. Discover Business Context & Strategic Focus
- Identify the founder's identity, company mission, core offerings, target geography, customer segments, and revenue model.
- Identify the founder's high-leverage strengths (strategy, sales, partnerships, product direction) vs. repetitive administrative drains to eliminate.

### 2. Map Functional Departments to Virtual Roles
Break the company into structured virtual responsibilities:
- **Founder's Office & Executive**: Chief of Staff, weekly retrospectives, daily priority triage.
- **Revenue & Business Development**: Account research, institutional prospect discovery, tailored proposal drafting, CRM tracking.
- **Product & Program Design**: Curriculum/service packaging, starter templates, hands-on lab sandboxes.
- **Partner & Talent Network**: Freelancer/trainer sourcing, skills vetting matrix, standard SOW/NDA generation.
- **Delivery Operations & Customer Success**: Logistics checklists, student/client feedback ingestion, automated certificate/outcomes generation.
- **Finance & Administration**: Program unit economics, invoice drafting, accounts receivable tracking, cash flow visibility.

### 3. Classify Work into 4 Autonomy Tiers
Establish strict human-in-the-loop boundaries for every recurring task:
- **Tier A — Fully Autonomous (Zero-Touch)**: Background data collection, syllabus diffing, file backups, scheduled data sync.
- **Tier B — AI-Prepared with Founder Approval (One-Click Gate)**: Outbound pitch emails, proposal documents, trainer contracts, social posts. AI drafts; founder reviews and sends.
- **Tier C — AI-Assisted Founder Work (Collaborative)**: Strategy memos, pricing models, curriculum architecture, client negotiation prep.
- **Tier D — Founder-Only (No Delegation)**: In-person meetings, MoU signatures, banking/payment authorization, high-stakes relationship management.

### 4. Audit Local Host Environment for Tool Compatibility
- Inspect the user's host OS (Windows, macOS, Linux) and installed runtimes (`node`, `python`, `git`, `docker`).
- Never recommend tools requiring Docker or Linux daemons if the environment lacks them. Prefer standalone executables (e.g., PocketBase `.exe`), Node CLI runners (`npx n8n`), and Python scripts.

### 5. Multi-Agent Control Plane & Autonomous Tooling
Integrate specific, production-proven agent control frameworks where Hermes acts as the CEO / central dispatcher:
- **Autonomous Company OS (Paperclip)**: Use Paperclip (`paperclipai/paperclip`) to model the company as an org chart with worker bots, task tickets, scheduled heartbeats, and hard monthly token/dollar budgets per agent to prevent runaway loops.
- **Agentic Research & Scientific Skills (Karpathy)**: Use `K-Dense-AI/karpathy` and the Scientific Agent Skills library (130+ skills) for deep literature review, academic paper synthesis, and technical benchmarking.
- **Multi-Agent Teams (TinyAGI / Edict)**: Decompose high-level company goals into subagent dependency graphs for parallel execution via `delegate_task`.

### 6. Select Native Open-Source & Local-First Stack
Prioritize tools that eliminate recurring monthly SaaS fees while keeping data owned locally:
- **Structured Database & CRM**: PocketBase (embedded SQLite, single `.exe` binary, instant web admin at `localhost:8090`).
- **Workflow Automation**: n8n (`npx n8n` / Node.js native, local SQLite).
- **Knowledge Base & SOPs**: Obsidian (local markdown, bi-directional wikilinks `[[...]]`).
- **Voice & Meeting Capture**: `whisper.cpp` / `obsidian-whisper-local` for offline voice-to-vault memo capture.
- **Scheduling**: Cal.com (open-source booking).
- **Contracts & E-Signatures**: Documenso (self-hostable digital signatures).
- **Privacy Analytics**: Umami (Node + SQLite).
- **Document Automation**: `python-docx` and `ReportLab` (programmatic Word/PDF generation).

### 7. Map End-to-End Business Workflows
Connect roles into closed-loop pipelines with explicit triggers, inputs, AI steps, approval gates, and outputs:
- **Revenue Workflow**: Prospect discovery → Lead enrichment in CRM → AI proposal draft → Founder approval → Outreach → Discovery call → MoU.
- **Delivery Workflow**: Client requirements → Service delivery preparation → Execution → Feedback form ingestion → Executive outcomes report.
- **Finance Workflow**: Service completion → Unit margin calculation → GST invoice drafting → Founder sign-off → AR tracking.

### 8. Document in Local Knowledge Base with Wikilinks
Store operating manuals, workflows, tool evaluation matrices, and SOPs in the user's local Obsidian vault using linked markdown notes (`[[Note Name]]`).

---

## Pitfalls

- Generating abstract, generic workflow diagrams when the user expects concrete, executable tools (like Paperclip, PocketBase, or Karpathy research skills).
- Assuming Docker or Linux containers exist without checking the actual host runtime; prefer standalone binaries and native Node/Python scripts on Windows.
- Allowing AI to autonomously send external messages, sign contracts, or initiate payments without an explicit founder approval gate.
- Recommending expensive monthly SaaS subscriptions when lightweight, self-hosted open-source tools can run on the host for $0.
- Failing to set hard spend budgets or scheduled heartbeats on autonomous agents, risking runaway API loops.
- Creating isolated prompt templates rather than connected end-to-end business workflows.
