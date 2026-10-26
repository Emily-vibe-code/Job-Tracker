# Architecture

The app is one HTML file: markup and CSS first, then a single script wrapped in an IIFE. There is
no framework and no build step. State lives in one object, and tables and drawers are re-rendered
from it.

## Data model

Three collections, each stored in the browser's local storage and, when available, in the artifact
database:

| Collection | Key | Notes |
|---|---|---|
| `jobs` | generated id | One record per posting: stage, dates, ratings, salary, notes, plus `interviews[]`, `followups[]` and `replies[]` (each `{id, date, notes}`) |
| `contacts` | generated id | Business/cell phone, business/personal email, LinkedIn, `replies[]`; optionally linked to a job through `jobId`, otherwise carries its own `company` text |
| `companies` | slug of the normalized name | Only what can't be derived: My rating, LinkedIn page, Company URL, General notes, a `manual` flag (added by hand), a `keep` flag (left behind by a deleted job), and copies of the five ratings for companies with no job |

Every record carries `createdAt` and `updatedAt`. `updatedAt` is bumped on every save.

## Companies are derived

There is no "add company" step in the data. `buildCompanyIndex()` groups jobs and contacts by
`companyKey(name)` (lowercased, punctuation and suffixes like Inc/LLC removed) and computes a
roll-up per company: the latest follow-up, application, interview and reply dates, contact count, and
every distinct PTO/Medical text. Ratings come from the most recently edited job that has them, taken
as a group so a rating is never paired with another job's review count.

A stored `companies` record is created only when something company-level is entered, or when a
company is added by hand, left behind by a deleted job, or created from a contact's Company field.
Merging companies renames the Company on every affected job and contact and folds the records
together.

## Sync

Cloud sync uses the Claude Artifacts `db` capability (`onSnapshot`, `get`, `set`). If it isn't
present the app runs on local storage alone.

- **Debounced writes.** Edits write after 500 ms, and a pending write is flushed when a drawer
  closes or a cell loses focus.
- **Never apply a stale snapshot over a live edit.** Incoming snapshots are ignored while a drawer
  is open, a cell is focused, or a write is scheduled or in flight (with a timeout so a dropped
  request can't block sync forever).
- **Merge, never delete.** A snapshot adds and updates records but never removes a local one just
  because the snapshot lacks it. Deletions only happen through an explicit delete.
- **Last write wins, by `updatedAt`.** When both sides have a record, the more recently touched one
  is kept.
- **Merge on write.** Before writing a record the app re-reads the cloud copy and merges field by
  field against the last version it saw. A field overrides the cloud only if this tab changed it, so an
  old open tab can't blank out newer data.

## Behaviours worth knowing

- **Stage transitions are field-driven.** *Date applied*, *Date rejected* and the *Interviewing*
  checkbox call small functions that decide the stage. A rejected job that still has a rejection date
  stays rejected even if an applied date is added.
- **Follow-up due.** `syncFollowUpDue()` runs inside `save()`. It stores the activity date it
  derived from (`followUpBasis`) so a manual edit sticks until new activity is logged.
- **Date cells.** Table date cells display MM/DD and swap to a real date input while focused; the
  year is always stored. Follow-up, Interview and Reply columns show and edit the most recent entry
  of a list.
- **Tables are built per tab** from column definitions; the Interviewing tab adds one column per
  interview logged.
- **Theme.** Colors are CSS variables with light and dark sets (`prefers-color-scheme`, plus an
  explicit `data-theme`).
