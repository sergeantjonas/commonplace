# Status page

**Status:** Being built in [`~/dev/status.vyoh.gg`](https://github.com/sergeantjonas/status.vyoh.gg); the implementation plan lives in that repo's `docs/working-notes/status-page.md`. This note holds only the part that spans projects.

## The box cannot watch itself

Anything that reports on the box has to keep working while the box is down. Otherwise it goes quiet in exactly the case it exists for. For `status.vyoh.gg` that plays out three ways:

- **The page is hosted off the box.** A status page served by the nginx it reports on shows nothing when that nginx is down. A static page on GitHub Pages under a `status.vyoh.gg` CNAME has no dependency on the box.
- **Alerts come from healthchecks.io, not from the notify hub.** The hub is likely to run as a tenant on this box ([notify-hub.md](notify-hub.md) § Open), so an alert that the box is down, routed through it, would never arrive. healthchecks.io already emails about the backup checks and can post to Discord without the hub.
- **Silence counts as a failure.** healthchecks.io alerts when a check stays quiet past its schedule plus grace. A prober that stopped running, or a box that took the prober down with it, therefore alerts the same way as a site that returned an error.

The same constraint applies to anything else that reports on the box's health, including the notify hub itself: whichever thing tells you the box is down cannot live on the box.
