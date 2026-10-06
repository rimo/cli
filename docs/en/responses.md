# Response reference

[English](../en/responses.md) | [日本語](../ja/responses.md)

What every `rimo` command returns, field by field. For the commands themselves
(syntax, flags, examples) see [Commands](commands.md); for the output contract
and error format see [Output & errors](output-and-errors.md).

## How to read this page

- **Presence** is either **Always** (the field is in every response) or
  **Optional**. An optional field is **omitted entirely** when it has no value —
  it is not returned as `null` or `""`, so check for the key's presence rather
  than comparing to an empty value. The one exception is
  [`Participant.user_id`](#participant), which is always present but is `""`
  for a speaker with no Rimo account.
- Timestamps are RFC 3339 strings in the caller's local timezone, for example
  `"2026-07-05T15:45:20.47178+09:00"`.
- Key order is not part of the contract — do not rely on it.

## What each command returns

| Command | Format | Shape |
|---------|--------|-------|
| [`rimo auth login`](commands.md#rimo-auth-login) | JSON | [Login and switch result](#login-and-switch-result) |
| [`rimo auth logout`](commands.md#rimo-auth-logout) | JSON | [Logout result](#logout-result) |
| [`rimo auth status`](commands.md#rimo-auth-status) | JSON | [Auth status](#auth-status) |
| [`rimo auth switch`](commands.md#rimo-auth-switch) | JSON | [Login and switch result](#login-and-switch-result) |
| [`rimo note list`](commands.md#rimo-note-list) | JSON | `{notes: `[Note](#note)`[], next_page_token}` |
| [`rimo note get`](commands.md#rimo-note-get) | JSON | `{note: `[Note](#note)`}` |
| `rimo note get --list-documents` | JSON | `{documents: `[Document](#document)`[]}` |
| `rimo note get --transcript` / `--document` / `--full` / `--meeting-chat` / `--document-id` | Plain text | [Note content](#note-content) |
| [`rimo note search`](commands.md#rimo-note-search) | JSON | `{notes: `[Search result](#search-result)`[], total_count}` |
| [`rimo note ask`](commands.md#rimo-note-ask) | Plain text | [Ask answer](#ask-answer) |
| [`rimo note create`](commands.md#rimo-note-create) | JSON | `{note: `[Note](#note)`, document: `[Document](#document)`}` |
| [`rimo note append`](commands.md#rimo-note-append) | JSON | `{document: `[Document](#document)`}` |
| [`rimo team list`](commands.md#rimo-team-list) | JSON | `{teams: `[Team](#team)`[], next_page_token}` |
| [`rimo version`](commands.md#rimo-version) | Plain text | One version line |
| [`rimo upgrade`](commands.md#rimo-upgrade) | Plain text | One status line |
| [`rimo mcp`](commands.md#rimo-mcp) | — | An MCP server on stdio, not a data command. See [MCP](mcp.md). |

`rimo login` and `rimo logout` are aliases for `rimo auth login` / `rimo auth
logout` and return the same shapes.

Any command can fail instead, in which case it prints an
[error object](output-and-errors.md#error-format) and exits 1.

## Envelopes

Every JSON response wraps its payload under a container key, alongside any
scalar metadata:

| Envelope | Returned by |
|----------|-------------|
| `{notes: [...], next_page_token: "..."}` | `note list` |
| `{notes: [...], total_count: <int>}` | `note search` |
| `{teams: [...], next_page_token: "..."}` | `team list` |
| `{note: {...}}` | `note get` |
| `{documents: [...]}` | `note get --list-documents` |
| `{note: {...}, document: {...}}` | `note create` |
| `{document: {...}}` | `note append` |
| `{accounts: [...], active_account: "..."}` | `auth status` |

`--fields` and `--excludes` apply to the **records inside** the envelope; the
scalar metadata siblings (`next_page_token`, `total_count`, `active_account`)
are always preserved. So `rimo note list --fields id,title` trims each note but
keeps `next_page_token`. See
[Field filtering](output-and-errors.md#field-filtering).

`next_page_token` is a cursor: pass it back as `--page-token` to fetch the next
page, and stop when it is absent.

## Note

Returned inside `{note: ...}` by `note get` and `note create`, and inside
`{notes: [...]}` by `note list`.

| Field | Type | Presence | Description |
|-------|------|----------|-------------|
| `id` | string | Always | The note ID. Pass it to `note get`, `note append`, and the other note commands. |
| `title` | string | Always | The note title. |
| `state` | string | Always | Where the note is in the recording/transcription pipeline. See [Note states](#note-states). |
| `duration` | integer | Always | Media length in **milliseconds**. `0` for notes with no recording. |
| `created_at` | string | Always | When the note was created. |
| `updated_at` | string | Always | When the note was last updated. Filter on this with `note list --updated-since`. |
| `user_id` | string | Optional | ID of the user who owns the note. |
| `organization_id` | string | Optional | ID of the organization the note belongs to. Absent for personal notes. |
| `team_id` | string | Optional | ID of the team the note belongs to. Absent for personal notes. Pass it to `note list --team`. |
| `locale` | string | Optional | Transcription language, e.g. `ja-JP`. |
| `media_type` | string | Optional | Kind of media attached, e.g. `audio`, `none`. |
| `source` | string | Optional | How the note was created, e.g. `none`, `zoom_import`. New values are added as integrations are added. |
| `held_at` | string | Optional | When the meeting took place. Filter on this with `note list --since` / `--until`. |
| `memo` | string | Optional | Free-text memo added to the note. |
| `share_mode` | string | Optional | Link sharing: `nothing` (not shared), `view`, or `edit`. |
| `custom_template_id` | string | Optional | ID of the custom template used for the minutes. |
| `webhook_url` | string | Optional | Webhook called when processing completes. |
| `document_markdown` | string | Optional | The primary document's markdown. See the note below. |
| `tags` | string[] | Optional | Tag names on the note. See the note below. |
| `participants` | [Participant](#participant)[] | Optional | Meeting participants. See the note below. |

**`note list` and `note get` return metadata only.** They never populate
`document_markdown`, `tags`, or `participants`, because loading a note's content
is the expensive part and most callers only need the metadata. To read content,
use the dedicated flags — `note get --document`, `--transcript`, `--full`,
`--list-documents` — which return [plain text](#note-content) or
[Document](#document) records.

### Note states

`state` tracks the note through recording, media conversion, and speech
recognition. Treat it as an open set: match the values you care about and handle
unknown ones gracefully, because new states are added as new capture methods
ship.

| State | Meaning |
|-------|---------|
| `NOTE_CREATED` | Note created with no media registered (e.g. by `note create`). |
| `NOT_SCHEDULED` | No bot recording scheduled. |
| `RECORDING_SCHEDULED` | Bot recording is scheduled. |
| `RECORDING_WAITING` | Bot joined the meeting and is waiting to start (e.g. host approval). |
| `RECORDING_STARTED` | Bot recording is in progress. |
| `RECORDING_PAUSED` | Bot recording is paused. |
| `RECORDING_DONE` | Bot recording finished. |
| `RECORDING_FAILED` | Bot recording failed and generally cannot be retried. |
| `MIC_RECORDING_REQUESTED` / `MIC_RECORDING_STARTED` / `MIC_RECORDING_DONE` | Microphone recording not yet started / in progress / finished. |
| `HARDWARE_RECORDING_REQUESTED` / `HARDWARE_RECORDING_STARTED` | Hardware recording not yet started / in progress. |
| `TRANSCRIPTION_DONE` | Real-time transcription finished, but the media has not been uploaded. |
| `MEDIA_PREPARING` | Audio or video is uploading. |
| `VC_WAITING` | Audio uploaded, waiting for permission to process. |
| `VC_REQUESTED` | Media conversion in progress. |
| `VC_ERROR` | Media conversion failed. |
| `ASR_PROCESSING` | Speech recognition in progress. |
| `ASR_ERROR` | Speech recognition failed. |
| `ASR_DONE` | Speech recognition finished — the transcript and minutes are ready. |

## Participant

A meeting participant. Returned inside a [Note](#note)'s `participants` array.

| Field | Type | Presence | Description |
|-------|------|----------|-------------|
| `user_id` | string | Always | The participant's Rimo user ID. Empty when the speaker has no Rimo account. Pass it to `note search --participant`. |
| `id` | string | Optional | Internal participant ID. |
| `name` | string | Optional | Display name. |
| `email` | string | Optional | Email address. |
| `calendar_name` | string | Optional | Attendee name from the calendar event, for notes created from a calendar. |
| `speaker_id` | string | Optional | Diarization speaker identifier, used to attribute transcript lines to this participant. |

## Document

A document (minutes) attached to a note. A note can have several — translations,
different templates — and `primary` marks the canonical one.

Returned inside `{documents: [...]}` by `note get --list-documents`, and inside
`{document: ...}` by `note create` and `note append`.

| Field | Type | Presence | Description |
|-------|------|----------|-------------|
| `id` | string | Always | The document ID. Pass it to `note append` and `note get --document-id`. |
| `note_id` | string | Always | ID of the note this document belongs to. |
| `title` | string | Always | The document title. |
| `primary` | boolean | Always | `true` for the note's canonical minutes document. |
| `created_at` | string | Always | When the document was created. |
| `updated_at` | string | Always | When the document was last updated. |
| `export_markdown` | string | Optional | The document body as markdown. |
| `locale` | string | Optional | Document language, e.g. `ja-JP`. |
| `category` | string | Optional | What kind of document it is, e.g. `agenda`, `summary`, `translation`. |
| `template_mode` | string | Optional | Template the document was generated from, e.g. `minutes`. |
| `custom_template_id` | string | Optional | ID of the custom template used. |

**`--list-documents` includes every document's full `export_markdown`**, so the
response grows with the note's content — a note with a few translations can run
to tens of kilobytes. When you only need the IDs and titles, drop the bodies:

```bash
rimo note get <note_id> --list-documents --excludes export_markdown
```

## Team

Returned inside `{teams: [...]}` by `team list`.

| Field | Type | Presence | Description |
|-------|------|----------|-------------|
| `id` | string | Always | The team ID. Pass it to `note list --team`, `note search --team`, and `note create --team`. |
| `name` | string | Always | The team name — this is the folder name shown in the Rimo app. |
| `category` | string | Optional | `"team"` for a team folder. `"organization"` marks your organization's own folder, returned only with `--include-organization`; its `id` is the organization ID. |
| `is_private_channel` | boolean | Always | `true` when only team members can read the team's notes. |
| `member_ids` | string[] | Always | User IDs of the team's members. |
| `created_at` | string | Always | When the team was created. |
| `updated_at` | string | Always | When the team was last updated. |
| `parent_id` | string | Optional | ID of the parent team. |
| `description` | string | Optional | The team description. |

## Search result

Returned inside `{notes: [...], total_count}` by `note search`. This is a
**search hit**, not a full [Note](#note) — it carries only what the search index
returns.

| Field | Type | Presence | Description |
|-------|------|----------|-------------|
| `id` | string | Always | The note ID. Pass it to `note get`. |
| `title` | string | Optional | The note title. |
| `held_at` | string | Optional | When the meeting took place. |
| `created_at` | string | Optional | When the note was created. |
| `owner_name` | string | Optional | Display name of the note's owner. `--mode=filter` only. |
| `snippet` | object | Optional | Matching excerpts. `--mode=filter` only — see below. |

In `--mode=filter`, `total_count` is the raw match count before permission
filtering — the whole result set rather than the current page, which is what
makes `--page` / `--per` navigation possible. In `--mode=semantic` it is the
number of unique notes returned.

**`snippet`** carries the matching excerpts, with the matched terms wrapped in
HTML tags for highlighting. Each key is an array of strings, and all four keys
are present — empty arrays where that part of the note did not match.

| Field | Type | Presence | Description |
|-------|------|----------|-------------|
| `transcripts` | string[] | Always | Excerpts from the transcript. |
| `headings` | string[] | Always | Excerpts from the note's headings. |
| `annotations` | string[] | Always | Excerpts from annotations. |
| `document_markdowns` | string[] | Always | Excerpts from the document body. |

## Authentication responses

### Login and switch result

Returned by `auth login` (and `rimo login`) and `auth switch`. Every field is
always present.

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | `logged_in` for `auth login`, `switched` for `auth switch`. |
| `alias` | string | The account alias, used with `--account`. |
| `email` | string | The signed-in email address. |
| `name` | string | Display name, falling back to the email and then the user ID. |
| `org` | string | Organization name, falling back to the org ID and then `Personal`. |

### Logout result

Returned by `auth logout` (and `rimo logout`). Every field is always present.

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | Always `logged_out`. |
| `alias` | string | The alias that was logged out. |
| `active_account` | string | The active account after the logout — empty if the one you logged out was active, since there is no auto-promotion. |

### Auth status

Returned by `auth status`.

| Field | Type | Presence | Description |
|-------|------|----------|-------------|
| `active_account` | string | Always | Alias of the active account. |
| `accounts` | object[] | Always | Every saved account — see below. |
| `active_credential` | string | Optional | Present only when an environment variable overrides the configured account: `env:RIMO_API_KEY` or `env:RIMO_TOKEN`. When it is present, that credential — not `active_account` — is what authenticates requests. |
| `api_key_hint` | string | Optional | Masked form of the key, present only with `env:RIMO_API_KEY`. |

Each entry in `accounts`:

| Field | Type | Presence | Description |
|-------|------|----------|-------------|
| `alias` | string | Always | The account alias, used with `--account`. |
| `name` | string | Always | Display name. |
| `org` | string | Always | Organization name. |
| `active` | boolean | Always | `true` for the active account. |
| `token_status` | string | Always | `valid`, `expiring_soon`, `expired`, or `unknown` (the expiry could not be determined). |
| `email` | string | Optional | The account's email address. |

`auth status` refreshes any expired or near-expiry token before reporting, so
`token_status` reflects live state rather than what was last written to the
credential store.

## Plain-text output

These commands print plain text on success because a JSON wrapper would only get
in the way. Errors are still JSON — see
[Output & errors](output-and-errors.md#error-format).

### Note content

`note get` with a content flag prints the note's content directly:

| Flag | Output |
|------|--------|
| `--transcript` | One line per transcript segment, as `Speaker: content`. Segments whose speaker could not be resolved print the content alone. Empty segments are skipped. |
| `--document` | The primary document as markdown, preceded by `# <title>`. |
| `--full` | The transcript, then a blank line, then the primary document. |
| `--document-id <id>` | One document's markdown, in the same form as `--document`. |
| `--meeting-chat` | One line per web-meeting chat message, as `[HH:MM] sender: text`. The `[HH:MM]` and `sender:` parts are dropped when unavailable, and empty messages are skipped. |
| `--timestamps` | With `--transcript` or `--full`, prefixes each transcript line with `[HH:MM:SS]` — time **elapsed from the start of the recording**, not wall-clock time. A segment with no start time stays unprefixed. |

A note with no transcript or no chat messages prints nothing and exits 0.

### Ask answer

`note ask` streams the answer as the model generates it, then a `Sources:` block
— the canonical citation surface, since the answer text carries no inline
citations — then a `Fetch a note:` block. Unusually, all three go to **stdout**,
so this is the one command whose stdout is not machine-readable. See
[`rimo note ask`](commands.md#rimo-note-ask) for a full example.

### Version and upgrade

Both print a single line to stdout — see [`rimo version`](commands.md#rimo-version)
and [`rimo upgrade`](commands.md#rimo-upgrade) for the exact strings.

## Dry-run output

`--dry-run` on a write command (`note create`, `note append`) sends no request.
It returns a **representative example** of the response shape with
`"dry_run": true` added:

```json
{
  "document": { "id": "doc_abc123", "primary": true, "...": "..." },
  "dry_run": true
}
```

The values are placeholders from the API schema, not a preview of your own
input — use it to check the response shape and confirm the command parses, not
to see what your note will contain.

`--dry-run` is rejected on read commands with
`dry-run not supported for read operations`.
