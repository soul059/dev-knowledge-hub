# Chapter 3: Azure Boards

## Overview
**Azure Boards** is a work-tracking service within Azure DevOps. It provides Kanban boards, backlogs, sprint planning, dashboards, and custom queries to plan, track, and discuss work across teams using Agile, Scrum, or CMMI processes.

---

## 3.1 Work Item Types by Process

```
┌──────── PROCESS TEMPLATES ─────────────────────┐
│                                                  │
│  AGILE (default):                                │
│  ├── Epic                                        │
│  │   └── Feature                                 │
│  │       └── User Story                          │
│  │           └── Task                             │
│  │               └── Bug                          │
│  │                                                │
│  SCRUM:                                           │
│  ├── Epic                                         │
│  │   └── Feature                                  │
│  │       └── Product Backlog Item (PBI)           │
│  │           └── Task                              │
│  │               └── Bug                           │
│  │                                                 │
│  CMMI:                                             │
│  ├── Epic                                          │
│  │   └── Feature                                   │
│  │       └── Requirement                           │
│  │           └── Task                               │
│  │               └── Bug / Change Request           │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## 3.2 Boards Workflow

```
┌─── KANBAN BOARD ──────────────────────────────┐
│                                                │
│  New  │  Active  │  Resolved  │  Closed        │
│───────│──────────│────────────│─────────        │
│       │          │            │                 │
│  ┌──┐ │  ┌──┐   │  ┌──┐     │  ┌──┐          │
│  │US│ │  │US│   │  │US│     │  │US│          │
│  │#5│ │  │#3│   │  │#2│     │  │#1│          │
│  └──┘ │  └──┘   │  └──┘     │  └──┘          │
│       │  ┌──┐   │            │                 │
│  ┌──┐ │  │US│   │            │                 │
│  │US│ │  │#4│   │            │                 │
│  │#6│ │  └──┘   │            │                 │
│  └──┘ │          │            │                 │
│       │          │            │                 │
│  WIP: ∞│  WIP: 3 │  WIP: 2   │  WIP: ∞       │
│                                                │
└────────────────────────────────────────────────┘
```

---

## 3.3 Sprint Planning

```
┌────── SPRINT WORKFLOW ─────────────────────┐
│                                             │
│  Product Backlog                            │
│  ┌────────────────────────────┐             │
│  │ US#1: Login page       8pt │             │
│  │ US#2: Dashboard        5pt │──┐          │
│  │ US#3: Settings         3pt │  │          │
│  │ US#4: Profile          2pt │  │          │
│  │ US#5: Notifications   13pt │  │          │
│  └────────────────────────────┘  │          │
│                                   │          │
│  Sprint 1 (2 weeks, 18pt cap)    │          │
│  ┌────────────────────────────┐  │          │
│  │ US#1: Login page       8pt │◄─┤          │
│  │ US#2: Dashboard        5pt │◄─┤          │
│  │ US#3: Settings         3pt │◄─┤          │
│  │ US#4: Profile          2pt │◄─┘          │
│  │ ──────────────────────     │             │
│  │ Total: 18pt ✅             │             │
│  └────────────────────────────┘             │
│                                             │
│  Sprint Burndown Chart:                     │
│  18│ *                                      │
│  15│   *                                    │
│  12│     *                                  │
│   9│       *                                │
│   6│         *                              │
│   3│           *                            │
│   0│             *   (ideal burndown)       │
│   ─┴──┬──┬──┬──┬──┬──                      │
│     D1 D3 D5 D7 D9 D10                     │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 3.4 Queries and Dashboards

```bash
# Work Item Query Language (WIQL) examples

# Find all active bugs assigned to me
SELECT [System.Id], [System.Title], [System.State]
FROM WorkItems
WHERE [System.WorkItemType] = 'Bug'
  AND [System.State] = 'Active'
  AND [System.AssignedTo] = @Me

# Find all user stories in current sprint
SELECT [System.Id], [System.Title], [Microsoft.VSTS.Scheduling.StoryPoints]
FROM WorkItems
WHERE [System.WorkItemType] = 'User Story'
  AND [System.IterationPath] = @CurrentIteration
```

### Dashboard Widgets

| Widget | Purpose |
|--------|---------|
| **Burndown** | Track work remaining in sprint |
| **Velocity** | Average story points per sprint |
| **Cumulative Flow** | See WIP and bottlenecks |
| **Query Results** | Display query as a table |
| **Chart for Work Items** | Pie/bar charts of work items |
| **Lead/Cycle Time** | Measure delivery speed |

---

## 3.5 Linking Work Items to Code

```
┌────── TRACEABILITY ────────────────────────┐
│                                             │
│  Work Item: US#42 "Add payment gateway"     │
│       │                                     │
│       ├──► Branch: feature/payment-42       │
│       │                                     │
│       ├──► Commits: "feat: payment #42"     │
│       │    (auto-linked via #42 in message)  │
│       │                                     │
│       ├──► Pull Request: PR#15              │
│       │    (linked during PR creation)       │
│       │                                     │
│       ├──► Build: Build#200                 │
│       │    (tracked by pipeline)             │
│       │                                     │
│       └──► Release: v1.5.0                  │
│            (deployment tracked)              │
│                                             │
└─────────────────────────────────────────────┘
```

```bash
# Link commits to work items using # syntax
git commit -m "feat: add Stripe integration #42"

# Multiple work items
git commit -m "fix: handle null payments #42 #43"
```

---

## 3.6 Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Board columns don't match states | Custom states not mapped | Map custom states in Board Settings |
| Cannot move item on board | WIP limit reached | Complete existing items first or raise limit |
| Sprint burndown looks wrong | Story points not set or tasks not updated | Update remaining work daily |
| Queries return no results | Wrong iteration path or area path | Check `@CurrentIteration` and Area Path |
| Can't see team's board | Wrong team context selected | Switch team in top navigation |

---

## Summary Table

| Concept | Key Points |
|---------|-----------|
| **Azure Boards** | Agile planning with Kanban boards, backlogs, sprints |
| **Process Templates** | Agile (User Story), Scrum (PBI), CMMI (Requirement) |
| **Kanban Board** | Visual workflow with WIP limits per column |
| **Sprints** | Fixed time boxes with velocity tracking and burndown |
| **Queries** | WIQL-based search for work items |
| **Traceability** | Link work items → branches → commits → PRs → builds |

---

## Quick Revision Questions

1. **What are the three process templates in Azure Boards?**
   > Agile (User Stories), Scrum (Product Backlog Items), and CMMI (Requirements). Each defines different work item types.

2. **What is a WIP limit?**
   > Work-In-Progress limit caps how many items can be in a Kanban column at once, preventing bottlenecks and overload.

3. **How do you link a commit to a work item?**
   > Include `#<ID>` in the commit message (e.g., `git commit -m "fix: resolve login bug #42"`).

4. **What is a burndown chart?**
   > A graph showing remaining work (story points) over the sprint duration, tracking whether the team will finish on time.

5. **What is velocity in Azure Boards?**
   > The average number of story points a team completes per sprint, used for capacity planning.

---

[⬅ Previous: Azure Pipelines](02-azure-pipelines.md) | [⬆ Back to Table of Contents](../README.md) | [Next: Azure Test Plans ➡](04-azure-test-plans.md)
