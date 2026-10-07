# Commonplace

**Status:** Index — what lives here and why. Read first.

Notes that belong to no single project. Each repo's own working notes hold its
arcs, its decisions and its ship log; this holds what spans repos before any one
of them owns it.

## What belongs here

- An idea more than one project would use, before code exists for it anywhere.
- A contract between projects: the shape one sends and another receives.
- Facts about shared infrastructure that no tenant owns.

What does not: anything one repo builds. Once work starts in a repo, its arc
note lives there beside the code, so the commit that ships it can update it too.
The note here shrinks to the part that still spans projects and links to the
rest. A note tracking one repo's progress from outside it goes stale on the
first commit that forgets it.

## The projects

- **vyoh.gg** — `~/dev/vyoh.gg`, notes in `docs/working-notes/`.
- **Engram** — `~/dev/engram`, engram.vyoh.gg, notes in `docs/working-notes/`.

Both run on the shared netcup box. Its topology lives in the `shared-vps` skill
and in vyoh.gg's `docs/working-notes/ops/` for now, though it belongs to neither.

## The notes

Every note opens with a `**Status:**` line; trust it over this list.

- `notify-hub.md` — idea. One place that delivers every project's alerts to
  Discord, with each project deciding when to send. Engram's episode alerts and
  vyoh.gg's wishlist price drops are the first two senders.
- `status-page.md` — idea. status.vyoh.gg and an alert when a site on the
  shared box stops answering, all of it running off the box it watches.
