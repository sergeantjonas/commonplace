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
- **notify.vyoh.gg** — `~/dev/notify.vyoh.gg`, the notify hub, notes in
  `docs/working-notes/`.

All of them run on the shared netcup box. Its topology lives in the `shared-vps` skill
and in vyoh.gg's `docs/working-notes/ops/` for now, though it belongs to neither.

## The notes

Every note opens with a `**Status:**` line; trust it over this list.

- `notify-hub.md` — being built in `~/dev/notify.vyoh.gg`; here lives only
  what earns an alert and the contract every sender posts to. Engram's episode
  alerts are the first sender, vyoh.gg's wishlist price drops the second.
- `status-page.md` — being built in `~/dev/status.vyoh.gg`; here lives only
  the cross-project constraint ("the box cannot watch itself") that the same
  problem imposes on anything else that reports on the box.
