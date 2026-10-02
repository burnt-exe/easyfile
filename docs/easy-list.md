# Easy List

Easy List is a local-first smart list, task and checklist module for the EasyFile suite.

## Core capabilities

- Multiple named lists/workstreams
- Quick add with inline syntax: `#tag`, `@owner`, `!high`, `!low`, `due:YYYY-MM-DD`
- Priority, status, due date, category, owner, tags and notes
- Search, status/priority/category filters and sorting
- Manual move up/down ordering
- Completion progress and workspace KPIs
- Templates for Tasks, Shopping, Project and Packing lists
- Seed demo data
- Bulk complete visible items
- CSV import/export
- JSON backup/restore
- Print-friendly output
- Browser LocalStorage autosave
- Responsive mobile layout
- Keyboard shortcut: Ctrl/Cmd + K focuses quick add
- Local Smart Assist checklist suggestions

## Storage and privacy

The browser workspace is stored under `easy.list.workspace.v1`. No account or server is required for the core module.

## Future integration contract

A later server-backed edition can add:
- authenticated cross-device sync,
- shared/team lists,
- audit history,
- notifications and reminders,
- Easy CRM/Inventory/Purchase Order deep links,
- Microsoft 365 / Google Workspace task sync,
- a same-origin AI endpoint for richer checklist generation.

No API keys should be embedded in the static client.
