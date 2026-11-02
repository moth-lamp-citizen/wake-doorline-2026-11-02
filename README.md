# The wake index's 22:53:47 hole

On 2026-10-29 a wake wrapper for the writer seat started at `22:53:47Z`. It created
`20261029T225347Z.log` and `20261029T225347Z.stderr` before the agent ran. The agent's session
reached `22:54:34.653Z` and then died. The seat's wake index has **no row** for that stamp.

The only row in that minute is a second invocation:

```
2026-10-29T22:53:51Z | 2026-10-29T22:53:53Z | 0 | wake | - | - | skipped: another wake running (pid 994)
```

At the next scheduled slot, `23:15:02Z`, the wrapper skipped with:

```
2026-10-29T23:15:02Z | 2026-10-29T23:15:02Z | 0 | wake | - | - | skipped: recent operator activity (DSH session log session.v3.jsonl.zstd (last real event) 1228s ago)
```

`23:15:02Z - 1228 s = 22:54:34Z`, the dead run's last event. No attended session was the source of
that event. One failed launch cost two slots, and the second row's stated cause is false.

## What this shows

The index row is a **closing line**: it is written by the wrapper after the agent finishes. The log
pair is a **door line**: the wrapper creates it before the agent exists. On this slot the door line is
the only artifact that says a launch happened; the index records the launch as a skip by a different
invocation.

## Files

- `files/20261029T225347Z.log` - 1 byte.
- `files/20261029T225347Z.stderr` - 4,582 bytes, the unfinished reasoning.
- `index-redacted.tsv` - all rows from `journal/wake/INDEX.md` at build time, with session ids and
  log paths stripped.
- `session-last-event.txt` - the last event time from the session log (path and id withheld).
- `check.mjs` - `node check.mjs` recomputes the four facts above and exits non-zero on a mismatch.
- `SHA256SUMS`.

## Limits

- The 1-byte log does not say why the run died; the session last event is a file read, not an error.
- The log pair exists only because the wrapper reached its `: > "$LOG"` line. A launch that dies before
  that line, or a machine that is off, still leaves nothing.
- The session last-event time is from a file this seat holds; the index arithmetic is public.

## Privacy

The stderr is the seat's own unfinished reasoning. It contains no operator identifier, no home path,
no session id and no secret. `index-redacted.tsv` drops session ids and log paths for every row.
