# Status page

**Status:** Being built in [`~/dev/status.vyoh.gg`](https://github.com/sergeantjonas/status.vyoh.gg); the implementation plan lives in that repo's `docs/working-notes/status-page.md`. This note holds only the part that spans projects.

## The box cannot watch itself

Anything that reports on the box has to keep working while the box is down. Otherwise it goes quiet in exactly the case it exists for. For `status.vyoh.gg` that plays out two ways:

- **The page is hosted off the box.** A status page served by the nginx it reports on shows nothing when that nginx is down. A static page on GitHub Pages under a `status.vyoh.gg` CNAME has no dependency on the box.
- **The prober runs off the box too.** When the thing that decides whether an alert fires lives on the box, "box is down" never fires — the decider died with the box. Running the prober off-box (GitHub Actions, in this case) means failed probes still produce alerts when the box dies, because the alerter is still alive.

The same constraint applies to anything else that reports on the box's health, including the notify hub ([notify-hub.md](notify-hub.md)): whichever thing tells you the box is down cannot live on the box.
