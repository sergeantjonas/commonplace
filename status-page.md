# Status page

**Status:** Idea, committed to — nothing built yet. It will be built in its own
repo, `status.vyoh.gg`, with the prober on the box first.

`status.vyoh.gg`: one page that says whether each public site on the shared box
answers, plus an alert when one stops. It watches every tenant and belongs to
none of them.

## The box cannot watch itself

Anything that reports on the box has to keep working while the box is down.
Otherwise it goes quiet in exactly the case it exists for.

- **The page is hosted off the box.** A status page served by the nginx it
  reports on shows nothing when that nginx is down. A static page on GitHub Pages
  or Cloudflare Pages, under a `status.vyoh.gg` CNAME, has no dependency on the
  box.
- **Alerts come from healthchecks.io, not from the notify hub.** The hub is
  likely to run as a tenant on this box ([notify-hub.md](notify-hub.md) § Open),
  so an alert that the box is down, routed through it, would never arrive.
  healthchecks.io already emails about the backup checks, and it can post to
  Discord without the hub.
- **Silence counts as a failure.** healthchecks.io alerts when a check stays
  quiet past its schedule plus grace. A prober that stopped running, or a box
  that took the prober down with it, therefore alerts the same way as a site
  that returned an error.

## Shape

Each public hostname gets one healthchecks.io check. Today that is `vyoh.gg`,
`api.vyoh.gg`, `engram.vyoh.gg` and `api.engram.vyoh.gg`. The checks sit in a
project of their own, apart from the tenants' backup checks. Every few minutes a
prober requests each hostname and pings that hostname's check with a success or
a `/fail`.

The page reads healthchecks.io's status badges in the browser. A badge's JSON
form returns the current status, and the endpoint answers
`Access-Control-Allow-Origin: *` (probed 2026-10-07), so the page needs neither
a backend nor a build step to stay current. Badges can be scoped by tag, so
giving each hostname a tag of its own gives it a badge of its own. Badge URLs
are read-only and reveal only the status.

- **A probe checks what a visitor would reach.** It requests the public
  hostname over TLS, separately over IPv4 and IPv6, and expects a specific
  status code. With a wrong A record and a right AAAA, a site serves nothing over
  IPv4 but still passes a check that tries only one address family.
- **API hostnames are probed on their health route**, so that a process
  running with an unreachable database counts as down. Engram's is `/ready`,
  which answers 503 in that case. vyoh.gg's is `/health`.
- **Each state is a word as well as a colour:** up, late, down.
- **The prober runs on the box**, as a systemd timer that reuses the backup
  heartbeat pattern from the `shared-vps` skill: the key in an env file, sent
  over stdin, and a script that always exits 0. That needs no new
  infrastructure, and a dead box still alerts through silence. Its blind spot is
  the network between the box and everyone else. The box holds its public
  addresses on `eth0`, so a request to its own hostname never leaves the box,
  and the netcup firewall and upstream routing go untested.

## Open

- **A second prober off the box**, if that blind spot ever matters. A scheduled
  GitHub Actions workflow or a Cloudflare Worker cron trigger tests the path
  visitors take, at the cost of a second thing to keep running. GitHub's
  scheduled runs are delayed and sometimes dropped under load, so their grace
  has to be wide enough that a late runner is not read as an outage.
- **Build or adopt.** Uptime Kuma and Gatus host themselves, so watching this
  box would take a second machine. UptimeRobot and Better Stack probe from
  outside and host a page, both with free tiers. Upptime runs on Actions and
  Pages. Each of these replaces part of the plan above, and none reuses the
  healthchecks.io account that already alerts on backups.
- **Certificate expiry.** Certbot renews unattended. A probe could fail its
  check when the certificate it is served comes within some number of days of
  expiry, which would catch a renewal that has been failing silently.
- **Whether backups belong on the page.** The nightly backup checks are already
  in healthchecks.io, but their badges sit in each tenant's own project. A public
  page that lists backup state also says more than whether a site answers.
