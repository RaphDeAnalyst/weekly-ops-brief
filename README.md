# Weekly Ops Brief

A single-file, browser-only tool that turns a task-tracker CSV into a concise weekly operations brief. It highlights the highest-attention work, overdue tasks, blockers, and items waiting on other people, with a rule-based next step for every blocked task.

**Live demo:** [Weekly Ops Brief](https://raphdeanalyst.github.io/weekly-ops-brief/)

## How to use

1. Open `index.html` in a modern browser. No installation, server, or internet connection is required.
2. Upload a `.csv`, paste CSV text, or choose **Load sample data**.
3. Set the **As of** date and choose **Build weekly brief**. The sample button builds the brief immediately.
4. Use **Copy brief as text** to copy a plain-text version, or **Print / Save as PDF** to print the brief.
5. Choose **Download sample CSV** to save an example tracker.

## CSV format

Headers are trimmed and case-insensitive. Required headers must be present. Dates use the calendar-valid `YYYY-MM-DD` format.

| Column | Required | Meaning / accepted values |
|---|---:|---|
| `task` | Yes | Task name; rows with an empty task are skipped. |
| `owner` | Yes | Person responsible for the task. |
| `due_date` | Yes | Due date in `YYYY-MM-DD`; invalid dates cause that row to be skipped. |
| `status` | Yes | `todo`, `in_progress`, `blocked`, `waiting`, or `done`. Unknown values are treated as `todo` and shown with a warning. |
| `priority` | No | `high`, `medium`, or `low`; missing or unknown values default to `medium`. |
| `blocker` | No | Blocker description used by the next-step rules. |
| `since` | No | Date the item became blocked or waiting, in `YYYY-MM-DD`. |
| `waiting_on` | No | Person, team, or organization the task is waiting on. |

CSV fields may be quoted; quoted commas, escaped quotes (`""`), and CRLF line endings are supported. A non-empty invalid `since` date also causes the row to be skipped. Blank lines are ignored. Extra columns are ignored.

The **Load sample data** button generates example dates relative to today. The included `sample.csv` is a downloadable-style static example dated November 16, 2026.

## Suggested next-step rules

Rules are checked in order; the first match wins. Text rules search the `blocker` field.

| Order | Match | Suggested next step |
|---:|---|---|
| 1 | `/approv\|sign.?off\|decision/` | Escalate: a decision is needed from the leader. |
| 2 | `/budget\|payment\|fund\|invoice\|cash/` | Escalate: funding or payment decision needed. |
| 3 | `/away\|leave\|unavailable\|sick\|travel/` | Reassign: the owner is unavailable. |
| 4 | `/vendor\|supplier\|delivery\|shipment\|courier/` | Follow up with the vendor today and agree a firm new date. |
| 5 | Stuck for 5 or more days | Escalate: stuck for 5+ days. |
| 6 | Overdue and blocked | Reschedule: reset the due date and tell stakeholders. |
| 7 | Default | Follow up with the owner today and ask for an unblock plan. |

The matching rule label is shown beside each blocked task's suggestion. The rules are a small editable JavaScript array in `index.html`; there are no AI or LLM calls.

## Attention scoring

The **Needs your attention** section ranks open tasks by descending score and shows the top three:

```text
min(days overdue × 2, 20)
+ min(days blocked × 2, 20)
+ priority points (high 6, medium 3, low 0)
+ 4 if blocked
+ 2 if waiting
```

Days overdue are counted only when a task is open. Blocked-day points are based on `since` for blocked tasks; missing `since` contributes zero blocked-day points. Ties are ordered by task name.

## Privacy

All parsing and brief generation happen locally in your browser. The app makes no network requests and uploads no data. The file input is read locally; clipboard and print actions are initiated only when you choose them.

## Limitations

- Dates must use `YYYY-MM-DD`; locale-specific date formats are not interpreted.
- This is a rule-based summary, not a project-management system. It cannot infer missing tracker context.
- Empty owners are displayed as **Unassigned**; empty blocker and waiting-on details are identified as unspecified.
- Printing is styled for one A4 portrait page for ordinary tracker sizes. Very large trackers or long descriptions may span additional pages.
- Browser clipboard permissions vary; if copying is unavailable, the app reports the error.

## How it was built

AI-assisted with GitHub Copilot, built as a work sample, logic reviewed and tested by me. The application is one self-contained HTML file using vanilla HTML, CSS, and JavaScript, with no libraries, CDNs, or network requests.
