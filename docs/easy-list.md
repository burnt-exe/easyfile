# Easy List — EasyFile Work-Management Engine

Easy List is the workflow layer for the EasyFile suite.

**Architecture:** Easy List → Easy Tasks → Easy Workflow → EasyFile modules.

## Implemented capabilities

- Multiple worklists with team/workspace metadata and members
- Task lifecycle: Backlog, To do, In progress, Blocked, Done
- Priority, due date, reminders, categories, owners, tags and notes
- Recurring tasks: daily, weekly and monthly
- Automatic creation of the next recurring task when the current occurrence is completed
- Subtasks with independent completion state
- Task dependencies with completion blocking when prerequisites are incomplete
- List, Kanban, Calendar and Activity views
- Browser notifications for reminders while the page/browser session is active
- Small local attachments (1 MB per attachment)
- Comments and local activity history
- Search, filtering, sorting and manual ordering
- Worklist progress and workspace KPIs
- Workflow templates for Project, Sales, Procurement and Service Delivery
- Seed workflow demo
- CSV import/export and JSON backup/restore
- Print-friendly list output
- LocalStorage persistence
- Ctrl/Cmd + K quick-add shortcut
- Local Smart Assist workflow suggestions
- Easy CRM contact linking through the existing `crm` LocalStorage dataset
- Easy Inventory product linking through the existing `easy.inventory.v2` LocalStorage dataset
- One-click workflow handoff creation for Quote, Purchase Order, Job Card and Sales Order

## Storage model

Primary workspace key:

`easy.list.workspace.v2`

The module migrates data from:

`easy.list.workspace.v1`

Core state:

```json
{
  "version": 2,
  "currentListId": "...",
  "lists": [],
  "items": [],
  "activity": [],
  "handoffs": []
}
```

Each task can include:

- `status`
- `priority`
- `due`
- `reminder`
- `repeat`
- `category`
- `owner`
- `tags`
- `notes`
- `subtasks[]`
- `dependencies[]`
- `attachments[]`
- `comments[]`
- `crmLink`
- `inventoryLink`

## EasyFile workflow handoff contract

Easy List writes the latest outbound handoff to:

`easyfile.workflow.handoff.v1`

and also retains handoff history inside the Easy List workspace.

Example:

```json
{
  "schema": "easyfile.workflow.handoff.v1",
  "id": "handoff-id",
  "createdAt": "ISO-8601",
  "source": {
    "module": "easy-list",
    "listId": "list-id",
    "taskId": "task-id"
  },
  "target": {
    "module": "quote"
  },
  "task": {
    "title": "Prepare customer commercial",
    "notes": "Commercial context",
    "category": "Sales",
    "owner": "Sales Team",
    "due": "2026-10-10",
    "tags": ["customer", "quote"]
  },
  "crm": {
    "id": 123,
    "name": "Customer contact",
    "company": "Customer Ltd",
    "email": "contact@example.com",
    "phone": "+27..."
  },
  "inventory": {
    "sku": "SKU001",
    "name": "Product",
    "cost": 100,
    "price": 150,
    "stock": 10
  }
}
```

## Target-module behaviour

Quote, Purchase Order, Job Card and Sales Order modules should:

1. Check the query string for `source=easy-list`.
2. Read `easyfile.workflow.handoff.v1`.
3. Validate `schema === "easyfile.workflow.handoff.v1"`.
4. Optionally validate the query-string `handoff` ID against the envelope `id`.
5. Pre-populate only fields that map safely to the target module.
6. Keep the user in control of final document values.
7. Preserve `source.listId`, `source.taskId` and handoff ID in the created record/document metadata.
8. Never delete the handoff automatically before a successful import.

Suggested mappings:

| Easy List | Quote | Purchase Order | Job Card | Sales Order |
| --- | --- | --- | --- | --- |
| task.title | reference / description | reference / description | job description | reference / description |
| task.notes | notes | notes | notes | notes |
| task.due | validity / follow-up | delivery date | due date | delivery date |
| crm | client | supplier/customer where appropriate | customer | customer |
| inventory | line item candidate | line item candidate | parts candidate | line item candidate |

## Current static-host boundary

The module is fully functional as a local-first work-management engine. Shared/team lists currently represent team metadata inside the local workspace; true multi-user real-time collaboration requires an authenticated backend or sync service.

Browser reminders are checked while Easy List is open. Reliable operating-system-level reminders when the site is closed require a service worker/push backend or external task/calendar integration.

Attachments are intentionally capped at 1 MB each because browser LocalStorage is not appropriate for large binary files. A production shared edition should move attachments to Easy Save/object storage and retain only metadata and secure references in the task record.

## Next integration phase

The highest-value suite integration work is to add the handoff consumer to:

- `easy-quote.html`
- `easy-purchase-order.html`
- `easy-job-card.html`
- `easy-sales-order.html`

Then add:

- Easy Invoice handoffs
- Easy Expenses approval tasks
- CRM follow-up creation from contact records
- Inventory reorder tasks generated from low-stock conditions
- Easy Save attachment storage
- authenticated team workspaces
- audit/version history
- server-backed reminder scheduling
- Microsoft 365 / Google Workspace task and calendar sync
- same-origin AI endpoint for richer workflow generation

No API keys should be embedded in the static browser client.
