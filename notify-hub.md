# Notify hub

**Status:** Live at `notify.vyoh.gg` since 2026-10-10, with Engram sending; built in [`~/dev/notify.vyoh.gg`](https://github.com/sergeantjonas/notify.vyoh.gg), whose `docs/working-notes/notify-hub.md` holds the implementation plan. This note holds only the part that spans projects: what earns an alert, and the contract every sender posts to.

More than one project wants to tell the owner something the moment it happens:
Engram, that a new episode of a followed show is ready, stuck, or late;
vyoh.gg, that a game on the wishlist dropped in price. Each could post to a
Discord webhook of its own. This note is the case for not doing that.

## What earns an alert

Only what happened without the owner. Anything the owner just did is already
known to them, so adding a series, starting a search, buying a game or
changing a setting never pings, and neither does what follows directly from
it. A new episode landing, a download stalling, a price falling: those happen
while nobody is looking, and they are the whole point.

## Deciding stays, delivering moves

Deciding whether something is worth an alert needs the project's own data. Only
Engram knows which shows are followed; only vyoh.gg knows the wishlist and its
price history. So the decision stays in the project.

Delivering needs none of it, and it is the same problem everywhere: a webhook
URL to keep secret, a message that must not arrive twice, a burst of ten that
should arrive as one. The hub solves those once.

## The contract

A project posts one message per thing worth saying, to `POST /messages` with
`authorization: Bearer <its secret>`:

```json
{
  "key": "ready@show:tvdb:392276/s03e0005",
  "title": "S03E05 is on disk",
  "body": "You're one behind.",
  "url": "https://engram.vyoh.gg/..."
}
```

`key` and `title` are required; `body` and `url` are not, and no other field
is taken:

| Field | Limit |
|---|---|
| `key` | at most 200 characters, no whitespace |
| `title` | at most 256 characters |
| `body` | at most 2000 characters |
| `url` | `http` or `https`, at most 2048 characters |

The hub answers `202 { "duplicate": false }` for something new and
`202 { "duplicate": true }` for a key it already has. A 401 or a 5xx means it
does not have the message, and the sender keeps it and tries again. A 400 means
the message itself is wrong and will be refused however often it is sent, so
the sender logs it and gives up on that one.

- **`(source, key)` is delivered once.** Posting it again is a no-op, so a
  sender can retry freely. That matters most for checks that run on a schedule:
  an overdue episode is overdue on every run, and a price that stays low stays
  low for a week. Engram's ingest is idempotent the same way, on
  `(source, source_event_id)`.
- **The key is the sender's to shape.** A price that rose and fell back should
  alert again, so the sender puts what changed into the key. The hub never
  guesses what counts as new.
- **`url` opens where the alert can be acted on**, inside the project that sent
  it.
- **Each source has its own secret, and the secret names the source.** The
  body carries no `source`, so a leaked Engram secret cannot post as vyoh.gg.
- **A sender keeps what it decided until the hub has taken it.** A scheduled
  check re-sends on its next run and the key absorbs the repeat; a one-off
  event, such as a Sonarr import, is lost unless its sender holds on to it.
  Engram does, in its `alert` table.

## Where it runs

A tenant of the shared box at `notify.vyoh.gg`, so it cannot carry the alert
that the box itself is down. [status-page.md](status-page.md) handles that one
off-box: its prober runs in GitHub Actions, and a failed POST to the hub
makes the workflow exit non-zero so GitHub's workflow-failure email arrives
instead.

## Senders

- **Engram — episode alerts for followed shows.** The first sender, live
  2026-10-10. Ready, stuck and overdue, for recently aired episodes only;
  ready is the one built so far. Scoped as an arc in
  Engram's `docs/working-notes/episode-alerts.md`.
- **vyoh.gg — wishlist price drops.** Not written up there yet. The price data
  it would read from is scoped in vyoh.gg's
  `docs/working-notes/steam/steam-library-economics.md`.
