# Notify hub

**Status:** Idea — nothing built, nothing committed to. Two senders named.

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

A project posts one message per thing worth saying:

```json
{
  "source": "engram",
  "key": "ready:show:tvdb:392276:3:5",
  "title": "S03E05 is on disk",
  "body": "You're one behind.",
  "url": "https://engram.vyoh.gg/..."
}
```

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

## Delivery

- **Webhook URLs live in the hub and nowhere else.** A Discord webhook URL is a
  credential — whoever holds it can post to the channel — so no project holds
  one, and none reaches a log.
- **Bursts collapse.** Messages from one source within a short window arrive as
  one: a season pack is ten imports.
- **Each source routes to its own channel**, so one project's alerts can be
  muted without the other's.
- **What was delivered is recorded**, so "did it tell me?" has an answer.

## Open

- **Build or adopt.** Apprise API delivers to Discord and a hundred other
  targets but keeps no record, so it cannot refuse a repeat; at most it is the
  sending step inside the hub. ntfy pushes to a phone app rather than to
  Discord. A small service with one Postgres table is the likelier shape.
- **Where it runs.** Its own tenant on the shared box, or a route inside an
  existing app. A route inside Engram would tie vyoh.gg's alerts to Engram's
  uptime.
- **How senders authenticate.** One secret per source, so a leaked Engram
  secret cannot post as vyoh.gg.
- **When the hub is down.** A scheduled check re-sends on its next run and the
  key absorbs the repeat. A one-off event, such as a Sonarr import, is lost
  unless its sender retries.

## Senders

- **Engram — episode alerts for followed shows.** Ready, stuck and overdue,
  for recently aired episodes only; the gaps the overdue check has to close
  first are in Engram's `docs/working-notes/integration-ideas.md`.
- **vyoh.gg — wishlist price drops.** Not written up there yet. The price data
  it would read from is scoped in vyoh.gg's
  `docs/working-notes/steam/steam-library-economics.md`.
