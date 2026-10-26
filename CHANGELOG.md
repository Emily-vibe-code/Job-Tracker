# Changelog

Built in conversation with Claude Code, feature by feature. Newest first.

## Contacts, replies and companies
- Reply field and column (most recent date), with multiple dated reply notes on jobs and contacts.
- Table date cells show MM/DD, or a dash when empty.
- Contacts: Business #, Cell #, Business email, Personal email (international phone formats).
- Companies tab: derived roll-up, My rating, add / rename / delete / merge, editable ratings that
  update every job at the company, slide-out with contacts, follow-ups and replies.
- Recommend % column.
- Paste-from-bookmarklet on the Companies tab.

## Follow-up and sync hardening
- Due date that schedules itself 7 days after the latest activity, with overdue coloring
  (orange, red, grey "consider closing").
- Field-by-field merge on every save, so an old open tab can't overwrite newer data.
- `updatedAt` last-write-wins merging for incoming cloud snapshots.
- Job description spacing cleanup on paste and blur.
- Removed the "Vet this job" button to save space.

## Core tracker
- Jobs, Applied, Interviewing, Rejected and Contacts tabs with inline editing, slide-out records,
  sortable and resizable columns, sticky header and Actions column.
- Field-driven stage moves.
- Cloud sync through the artifact database with a local-storage fallback.
- Add-from-page bookmarklet, JSON backup export, dark mode.
