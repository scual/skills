# Issue tracker: ticgit

Issues for this repo live as Git-native tickets managed by the `ti` CLI. Tickets are stored as Git metadata (not `.scratch/` markdown, not GitHub/GitLab Issues) and travel with the repo via `ti sync`.

## Conventions

- **Create a ticket**: `ti new --title "..." [--tags ...] [--comment "..."]`. Don't create GitHub/GitLab issues or `.scratch/` markdown files.
- **Read tickets**: `ti list --markdown`, `ti show <id> --markdown`, or `ti list --json` / `ti show <id> --json` for programmatic parsing. Prefer `--markdown` when reading for context, `--json` when parsing fields.
- **Claim/work on a ticket**: `ti checkout <id>` to select a current ticket, `ti comment "..."` for progress notes.
- **Change lifecycle**: `ti state <status[:state]>` (e.g. `ti state blocked`, `ti state closed:wontfix`). `ti close` is shorthand for marking resolved.
- **Labels**: `ti tag <tags...> [-t <id>]` / `ti tag -d <tags...> [-t <id>]` to remove. See `triage-labels.md` for the role-to-tag mapping.
- **Assign**: `ti assign <user> [-t <id>]`.
- **Sync**: `ti sync` (pulls then pushes ticket metadata via Git). Ticket history moves with normal Git operations since it's stored in Git metadata, not a separate server.
- **No PR surface**: ticgit has no concept of pull requests as a triage input. Skills that pull external PRs into triage (GitHub/GitLab mode) don't apply here.

## Sequence & Ordering Conventions

- **Human Indexing**: Ticket titles should include a sequential zero-padded prefix when part of an ordered plan (e.g., `ti new --title "[01] Core parser"`).
- **Dependency Graphs**: Always link sequential steps with `ti depends <previous-id> -t <next-id>`.
- **Map Ledger**: When creating sub-issues for a `wayfinder-map`, record their sequential order in the map ticket's body under an `## Execution Plan` section with both order index and short hash.

## When a skill says "publish to the issue tracker"

Run `ti new` with the appropriate title, tags, and comment.

## When a skill says "fetch the relevant ticket"

Run `ti show <id> --markdown` (or `--json` if the skill is parsing fields). The user will normally pass the ticket id or a unique prefix directly.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a ticket; its tickets are **sub-issues** of the map.

- **Map**: `ti new --title "<effort name>" --tags wayfinder-map`, body holds the Destination / Execution Plan / Notes / Decisions-so-far / Not yet specified / Out of scope sections (edit via `ti edit <id>`).
- **Child ticket**: `ti new --title "<number> <question>" --subissue <map-id> --tags wayfinder-<type>` where `<type>` is `research`/`prototype`/`grilling`/`task`.
- **Blocking**: `ti depends <blocker-id> -t <ticket-id>` (native dependency — shows in `ti show`/`ti tui`). A ticket is unblocked when every dependency is closed.
- **Frontier**: `ti list --tag wayfinder-<map-id-or-tag> --json`, filtered to open, unassigned tickets whose `depends_on` are all closed. `ti next` also skips tickets with unresolved dependencies and can suggest the next one directly.
- **Claim**: `ti assign <own-email> -t <ticket-id>` before any work — an assigned open ticket, or subticket, is claimed.
- **Resolve**: `ti comment -t <ticket-id> "<answer>"` (or `ti edit` to fold the answer into the description), then `ti close -t <ticket-id>` (or `ti state resolved -t <ticket-id>`), then append a context pointer (gist + `ti show <id>` reference) to the map ticket's Decisions-so-far via `ti edit <map-id>`.
