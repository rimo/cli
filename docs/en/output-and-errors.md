# Output & errors

[English](../en/output-and-errors.md) | [日本語](../ja/output-and-errors.md)

## JSON-first by design

`rimo` prints JSON to stdout by default — and it does not matter whether you are
in a terminal, in a pipe, or inside CI. The output of a command is the same
everywhere, which is what makes it safe to script and safe for an AI agent to
parse. Nothing about your environment changes the format; only a flag does.

When you want to *read* the output rather than parse it, add `--pretty` — see
[Human-readable output](#human-readable-output---pretty) below.

A few human-facing commands print **plain text** on success instead, because a
JSON wrapper would just get in the way:

- `rimo version` and `rimo upgrade`
- `rimo note ask` (the streamed answer)
- `rimo note get` with `--transcript`, `--document`, `--full`, `--meeting-chat`,
  or `--document-id`

Even for these, **errors are always JSON**, so failures stay machine-readable.

Whichever the format, **stdout carries the data and stderr carries everything
else** — progress spinners, update notices, and the `Fetch a note:` /
pagination hints — so piping stdout into `jq` is always safe.

For what each command returns, field by field, see the
[Response reference](responses.md).

## Human-readable output (`--pretty`)

`--pretty` renders the same data as a table (for lists) or an aligned key/value
block (for a single record) instead of JSON.

```bash
rimo note list --pretty
```

```
ID                    TITLE                            HELD AT           DURATION  STATE
J9yyjDQLJWqhiSTH0sAT  週次定例 プロダクトチーム        2026-09-05 14:30  58m       ASR_DONE
aK2mLp0QQzXcVbNm1234  Rimo CLI design review — outpu…  2026-09-04 10:00  1h 05m    ASR_DONE

2 of 137 notes
Next page: --page-token next-page-2
```

It is **opt-in and never automatic**. `rimo note list` gives JSON whether or not
a terminal is attached, so a script that worked yesterday keeps working; and
`rimo note list --pretty | grep 定例` gives you exactly what you saw on screen.

### What it does with long values and narrow terminals

The layout adapts to your terminal width (taken from `COLUMNS` if you set it,
otherwise from the terminal itself, otherwise 100 columns when piped):

- **IDs and timestamps are never shortened.** An ID you cannot paste back into
  `rimo note get` would defeat the point, so long titles give up room instead.
- **When space runs out, whole columns are dropped**, least important first,
  rather than every column being shaved down to ellipses. Resize your terminal
  and re-run to see more.
- **Very long text fields** (a document's markdown, a search snippet, a memo)
  are never shown as columns — they would fill the row with a single `…`. Ask
  for them explicitly with `--fields` if you want them.
- **Multi-line values are flattened to one line** so a memo cannot break the
  table. To read one in full, fetch the single record: `rimo note get <id>
  --pretty` wraps long values instead of truncating them.

### Choosing columns

`--fields` picks the columns and their order; `--excludes` removes them:

```bash
rimo note list --pretty --fields id,title,held_at
rimo team list --pretty --excludes description
```

Columns you name with `--fields` are never dropped, even on a narrow terminal.

### When not to use it

- **In scripts and pipelines** — parse the JSON. The table's spacing and column
  set are a presentation detail and may change between releases.
- **For AI agents** — JSON is smaller, unambiguous, and combines with `--fields`
  to keep responses cheap.
- `--fields=compact` is a JSON-only mode (it swaps long values for
  `"[omitted]"`); with `--pretty` it is ignored, since the layout already fits
  values to your terminal.

Errors are **always JSON**, with or without `--pretty`, so failure handling
never depends on how the successful output is rendered.

## Field filtering

`--fields` and `--excludes` apply to any command whose output is JSON. They
filter the records *inside* the response
[envelope](responses.md#envelopes) — the scalar metadata siblings such as
`next_page_token` and `total_count` are always preserved.

### `--fields`

| Value | Behavior |
|-------|----------|
| `""` (default) | All fields, full values |
| `"compact"` | All fields; long string values replaced with `"[omitted]"` |
| `"f1,f2,f3"` | Only these fields |

```bash
rimo note list --fields compact
rimo note list --fields id,title,created_at
```

### `--excludes`

Comma-separated field names to remove from output. Applied after `--fields`.

```bash
rimo note list --excludes transcript,document_markdown
```

Field filtering is especially useful for AI agents — request only the fields you
need (`--fields id,title`) to keep responses small and cheap to parse.

## Dry runs

`--dry-run` on a write command sends no request and returns a placeholder
example of the response shape with `"dry_run": true` added. See
[Dry-run output](responses.md#dry-run-output).

## Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Error |

The exit code is the reliable signal of success vs failure — check it in scripts
rather than parsing output.

## Error format

Any error is written as JSON to stdout and the process exits with code 1:

```json
{
  "code": "error",
  "message": "unknown flag: --bogus"
}
```

| Field | Description |
|-------|-------------|
| `code` | Machine-readable error code. Currently always `error`. |
| `message` | Human-readable description of the failure (validation message, API status, etc.). |
