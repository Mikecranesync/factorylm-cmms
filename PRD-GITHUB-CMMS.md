# PRD: GitHub-Based CMMS System
## Product Requirements Document
**Version:** 1.0  
**Date:** 2026-02-10  
**Author:** Jarvis (AI) + Mike H  
**Status:** Draft for Review

---

## Executive Summary

Replace the fragile Atlas CMMS (Docker/PostgreSQL) with a **100% GitHub/Gist-based CMMS** that leverages GitHub's infrastructure for reliability, search, and collaboration. Technicians interact via gist comments on job plans - a workflow that matches how real maintenance work happens.

---

## Problem Statement

### Current State (Atlas CMMS)
- Docker container dependency → single point of failure
- PostgreSQL database → requires maintenance, backups
- Custom API → another failure point
- Last night's YC demo: **CMMS down = entire workflow broken**
- Complex deployment → hard to debug, hard to scale

### Desired State (GitHub CMMS)
- **Zero infrastructure** (GitHub handles everything)
- **99.9% uptime** (GitHub's SLA)
- **Built-in search, history, audit trail**
- **Mobile-ready** (GitHub app)
- **AI-searchable metadata**
- **Technician-friendly** (comment on job plans)

---

## User Stories

### Technician (End User)
1. *"As a tech, I want to receive a job plan with all details so I can work independently"*
2. *"As a tech, I want to update progress by adding comments so my supervisor knows status"*
3. *"As a tech, I want to attach photos of completed work as proof"*
4. *"As a tech, I want to see the full history of an equipment's repairs"*

### Supervisor/Manager
1. *"As a supervisor, I want a kanban view of all work orders"*
2. *"As a supervisor, I want to assign work to specific technicians"*
3. *"As a supervisor, I want to search past repairs by equipment or failure mode"*
4. *"As a supervisor, I want cost/time analytics on maintenance"*

### AI Agent (Jarvis/FactoryLM)
1. *"As an AI, I want to create work orders from photos via API"*
2. *"As an AI, I want to query metadata for analytics"*
3. *"As an AI, I want to monitor comments and respond to questions"*
4. *"As an AI, I want to update job plan status automatically"*

---

## Architecture

### Two-Layer System

```
┌─────────────────────────────────────────────────────────────────┐
│                     LAYER 1: GITHUB ISSUES                      │
│                   (Workflow & Collaboration)                    │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Issue = Work Order                                      │   │
│  │  ├── Title: WO-047 - Conveyor #3 VFD Replacement        │   │
│  │  ├── Labels: [priority-high] [conveyor] [electrical]    │   │
│  │  ├── Assignee: @tech-mike                               │   │
│  │  ├── Milestone: Week 6 PM                               │   │
│  │  ├── Body: Job plan (linked to gist)                    │   │
│  │  └── Comments: Tech updates, photos, completion notes   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Project Board (Kanban):                                        │
│  ┌──────────┬─────────────┬──────────────┬──────────┐          │
│  │ Backlog  │ In Progress │ Waiting Parts│ Complete │          │
│  └──────────┴─────────────┴──────────────┴──────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Links to
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     LAYER 2: GIST DATABASE                      │
│                   (Structured Metadata + Job Plans)             │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Gist: Job Plan (Full Technical Details)                 │   │
│  │  ├── job-plan.md: Step-by-step instructions             │   │
│  │  ├── wiring-diagram.md: Technical diagrams              │   │
│  │  ├── parts-list.md: Required materials                  │   │
│  │  └── Comments: Tech questions & AI answers              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Gist: Metadata Store (AI-Searchable JSON)               │   │
│  │  {                                                       │   │
│  │    "WO-047": {                                           │   │
│  │      "equipment_id": "CONV-003",                         │   │
│  │      "failure_mode": "overheating",                      │   │
│  │      "parts_used": ["GS11N-20P2"],                       │   │
│  │      "labor_hours": 2.5,                                 │   │
│  │      "cost": 285.00                                      │   │
│  │    }                                                     │   │
│  │  }                                                       │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Data Flow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   TELEGRAM   │────▶│    JARVIS    │────▶│    GITHUB    │
│  (Input)     │     │  (AI Layer)  │     │   (Storage)  │
└──────────────┘     └──────────────┘     └──────────────┘
                            │
                     ┌──────┴──────┐
                     ▼             ▼
              ┌──────────┐  ┌──────────┐
              │  ISSUE   │  │   GIST   │
              │ (Track)  │  │ (Detail) │
              └──────────┘  └──────────┘
```

---

## Components

### 1. Work Order Repository
**Repo:** `Mikecranesync/factorylm-cmms`

**Structure:**
```
factorylm-cmms/
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   └── work-order.yml        # WO template
│   └── workflows/
│       └── wo-automation.yml     # Auto-labeling, notifications
├── docs/
│   └── equipment/                # Equipment documentation
├── README.md                     # CMMS overview
└── LABELS.md                     # Label taxonomy
```

**Labels:**
| Category | Labels |
|----------|--------|
| Priority | `priority-critical`, `priority-high`, `priority-medium`, `priority-low` |
| Equipment | `conveyor`, `pump`, `motor`, `plc`, `vfd`, `hvac` |
| Type | `corrective`, `preventive`, `inspection`, `project` |
| Status | `waiting-parts`, `waiting-approval`, `scheduled` |
| Location | `line-1`, `line-2`, `warehouse`, `utilities` |

### 2. Job Plan Gists
Each complex work order gets a dedicated gist:

**Gist Structure:**
```
WO-047-conveyor-vfd/
├── README.md           # Job plan overview
├── procedure.md        # Step-by-step instructions
├── wiring-diagram.md   # Technical diagrams (ASCII/Mermaid)
├── parts-list.md       # BOM with part numbers
├── safety.md           # LOTO, PPE requirements
└── metadata.json       # Structured data for AI
```

**Comment Workflow:**
1. Tech receives notification → Opens gist
2. Tech reads procedure → Asks clarifying question in comment
3. AI (Jarvis) monitors → Responds with answer
4. Tech completes step → Adds comment "Step 3 complete, found corrosion on terminal 4"
5. AI updates metadata → Logs finding for future reference
6. Tech attaches photo → Proof of completion
7. Job complete → AI closes issue, updates analytics

### 3. Metadata Gist (AI Database)
**Single gist as key/value store:**

```json
{
  "equipment": {
    "CONV-003": {
      "name": "Conveyor Line 3",
      "manufacturer": "Dorner",
      "install_date": "2019-03-15",
      "location": "line-2",
      "criticality": "high",
      "work_orders": ["WO-012", "WO-031", "WO-047"]
    }
  },
  "work_orders": {
    "WO-047": {
      "equipment_id": "CONV-003",
      "type": "corrective",
      "failure_mode": "vfd_overheating",
      "root_cause": "capacitor_degradation",
      "parts": [{"pn": "GS11N-20P2", "qty": 1, "cost": 245}],
      "labor_hours": 2.5,
      "downtime_hours": 4,
      "created": "2026-02-10T08:00:00Z",
      "closed": "2026-02-10T12:30:00Z",
      "gist_id": "abc123...",
      "issue_number": 47
    }
  },
  "analytics": {
    "mtbf": {"CONV-003": 720},
    "mttr": {"CONV-003": 2.1},
    "costs_mtd": 1250.00
  }
}
```

### 4. Integration Scripts

**Location:** `/opt/factorylm/github-cmms/`

| Script | Function |
|--------|----------|
| `create_wo.py` | Create issue + gist from Telegram photo |
| `monitor_gists.py` | Poll gist comments, respond via AI |
| `sync_metadata.py` | Update metadata gist on issue close |
| `search_wo.py` | Query metadata for analytics |
| `migrate_atlas.py` | One-time migration from Atlas |

---

## Atlas CMMS Archival Plan

### Phase 1: Freeze (Immediate)
1. Stop accepting new work orders in Atlas
2. Export all data to JSON backup
3. Document current state

### Phase 2: Migrate (This Week)
1. Convert existing WOs to GitHub issues
2. Create gists for active job plans
3. Build metadata gist from Atlas DB

### Phase 3: Sandbox (Archive)
1. Move Atlas code to `/archive/atlas-cmms/`
2. Add `ARCHIVED.md` explaining status
3. Keep Docker compose for reference only
4. Remove from active deployment

### Phase 4: Deprecate (30 Days)
1. Verify GitHub CMMS working fully
2. Final Atlas backup
3. Remove Atlas containers
4. Delete Atlas database (keep backup)

**Archival Structure:**
```
factorylm-monorepo/
├── archive/
│   └── atlas-cmms/
│       ├── ARCHIVED.md          # Why archived, how to restore
│       ├── docker-compose.yml   # For reference
│       ├── src/                  # Original source
│       ├── backups/
│       │   └── 2026-02-10.sql   # Final DB dump
│       └── migration-log.md     # What was migrated where
├── packages/
│   └── github-cmms/             # NEW: Active CMMS
└── ...
```

---

## Implementation Plan

### Week 1: Foundation
| Day | Task | Deliverable |
|-----|------|-------------|
| 1 | Create `factorylm-cmms` repo | Repo URL |
| 1 | Set up labels, templates | Label system |
| 1 | Create project board | Kanban board |
| 2 | Initialize metadata gist | Gist ID |
| 2 | Build `create_wo.py` | Script working |
| 3 | Build `monitor_gists.py` | Comment monitoring |
| 3 | Test end-to-end workflow | Demo video |

### Week 2: Migration
| Day | Task | Deliverable |
|-----|------|-------------|
| 1 | Export Atlas data | JSON backup |
| 2 | Run migration script | Issues created |
| 3 | Verify data integrity | Checklist |
| 4 | Archive Atlas code | Archive complete |
| 5 | Update FactoryLM bot | Bot uses GitHub |

### Week 3: Polish
| Day | Task | Deliverable |
|-----|------|-------------|
| 1-3 | Real-world testing | Bug fixes |
| 4 | Documentation | User guide |
| 5 | Analytics dashboard | Basic metrics |

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Uptime | 99.9% | GitHub status |
| WO Creation Time | <30s | Telegram → Issue |
| Tech Adoption | 100% | Comments on gists |
| Search Response | <2s | Query metadata |
| Data Loss | 0% | All Atlas data migrated |

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| GitHub API rate limits | Can't create WOs | Batch operations, caching |
| Gist size limits (100 files) | Can't store all data | Multiple metadata gists |
| No real-time webhooks on gists | Delayed responses | Polling + Issues for notifications |
| Learning curve for techs | Low adoption | Training, simple workflow |

---

## Appendix A: API Reference

### Create Work Order
```python
def create_work_order(photo_path, description, priority="medium"):
    # 1. Analyze photo with Gemini
    analysis = analyze_equipment_photo(photo_path)
    
    # 2. Create gist with job plan
    gist = gh.create_gist(
        description=f"WO-{next_id}: {analysis.equipment}",
        files={
            "README.md": job_plan_template(analysis),
            "metadata.json": analysis.to_json()
        }
    )
    
    # 3. Create GitHub issue
    issue = gh.create_issue(
        repo="Mikecranesync/factorylm-cmms",
        title=f"WO-{next_id}: {analysis.title}",
        body=f"**Job Plan:** {gist.url}\n\n{analysis.summary}",
        labels=[f"priority-{priority}", analysis.equipment_type]
    )
    
    # 4. Update metadata gist
    update_metadata(next_id, analysis, gist.id, issue.number)
    
    return {"issue": issue.url, "gist": gist.url}
```

### Monitor Gist Comments
```python
def monitor_gist_comments(gist_id, last_check):
    comments = gh.get_gist_comments(gist_id)
    new_comments = [c for c in comments if c.created_at > last_check]
    
    for comment in new_comments:
        if comment.user != "jarvis-bot":
            # AI responds to tech questions
            response = ai_respond(comment.body, gist_context)
            gh.create_gist_comment(gist_id, response)
            
            # Update job plan if needed
            if should_update_plan(comment):
                update_gist_file(gist_id, "procedure.md", new_content)
```

---

## Appendix B: Workflow Diagram

```
TECHNICIAN                    JARVIS                      GITHUB
    │                           │                           │
    │  📷 Send photo            │                           │
    │ ─────────────────────────▶│                           │
    │                           │  🔍 Analyze image         │
    │                           │ ─────────────────────────▶│
    │                           │                           │
    │                           │  📝 Create Issue + Gist   │
    │                           │ ─────────────────────────▶│
    │                           │                           │
    │  📬 Receive notification  │◀─────────────────────────│
    │◀─────────────────────────│                           │
    │                           │                           │
    │  💬 Comment on gist       │                           │
    │ ─────────────────────────────────────────────────────▶│
    │                           │                           │
    │                           │  👀 Detect new comment    │
    │                           │◀─────────────────────────│
    │                           │                           │
    │  🤖 AI response           │                           │
    │◀─────────────────────────│  📝 Post response         │
    │                           │ ─────────────────────────▶│
    │                           │                           │
    │  ✅ Mark complete         │                           │
    │ ─────────────────────────────────────────────────────▶│
    │                           │                           │
    │                           │  📊 Update metadata       │
    │                           │ ─────────────────────────▶│
    │                           │                           │
    │                           │  🔒 Close issue           │
    │                           │ ─────────────────────────▶│
```

---

**END OF PRD**

*Ready for review. Say "APPROVED" to begin implementation.*
