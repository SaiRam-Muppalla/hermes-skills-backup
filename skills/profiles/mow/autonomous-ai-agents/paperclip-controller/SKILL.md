---
name: paperclip-controller
description: Use when managing Paperclip companies and agent tickets.
version: 1.0.0
author: Makers on Wheels
tags: [Paperclip, Multi-Agent, Management, Orchestration]
---

# Paperclip Controller Skill

Use this skill to interact with the local Paperclip multi-agent orchestrator.

## Server Endpoint
Default local endpoint: `http://localhost:3100`

## Common Actions

### 1. Check Server Status
```bash
curl -s http://localhost:3100/api/health || echo "Paperclip server is not running"
```

### 2. List Companies
```bash
curl -s http://localhost:3100/api/companies
```

### 3. Fetch Open Tickets
```bash
curl -s "http://localhost:3100/api/companies/{companyId}/tickets?status=open"
```

### 4. Create a Task Ticket for Worker Agents
```bash
curl -X POST "http://localhost:3100/api/companies/{companyId}/tasks" \
  -H "Content-Type: application/json" \
  -d '{"role":"researcher","goal":"Research top 5 engineering colleges in Visakhapatnam","budget_limit_usd":5.0}'
```
