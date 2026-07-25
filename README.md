# Ted Dashboard

YAML-mode Home Assistant dashboard (`ted-dashboard`), versioned in git and deployed to a
live instance with the **Git pull** add-on (Pattern B: git-first, pull-to-deploy).

## How it maps to Home Assistant

This repository's **root maps onto `/config`**. The Git pull add-on clones/pulls it into
`/config`, so files land like this:

```
<repo root>            -> /config
  dashboards/
    ted-dashboard.yaml           -> /config/dashboards/ted-dashboard.yaml   (entry)
    ted-dashboard/               -> /config/dashboards/ted-dashboard/        (partials)
      kiosk.yaml
      navbar.yaml                (shared Ted's Navbar — single source)
      view-welcome.yaml
      view-home-nightstand.yaml
      view-home-wallpanel-h.yaml
      view-home-wallpanel-v.yaml
      view-home-handheld.yaml
      view-cameras.yaml
      view-assist-response.yaml
      view-climate.yaml
      view-weather.yaml
      view-music.yaml
      view-calendar-month.yaml
      view-calendar-week.yaml
      view-mediaroom.yaml
      view-alarms-timers.yaml
      view-settings.yaml
```

Only `dashboards/**` is tracked; everything else in `/config` (secrets, `.storage`, the
database) stays local and untracked (see `.gitignore`).

## Wiring it up (one-time, on Home Assistant)

Add to `/config/configuration.yaml` (this file is **not** in the repo):

```yaml
lovelace:
  dashboards:
    ted-dashboard:
      mode: yaml
      filename: dashboards/ted-dashboard.yaml
      title: Ted Dashboard
      icon: mdi:tablet
      show_in_sidebar: true
```

Then restart Home Assistant once. Afterwards, editing dashboard files needs only a browser
refresh (no restart).

## `!include` layout

`!include` paths resolve relative to the **directory of the file containing the directive**.
`ted-dashboard.yaml` includes the partials from `ted-dashboard/`, which is organized into:

- `shared/` — reused partials: `kiosk.yaml`, `navbar.yaml`, `navbar-autohide.yaml`,
  `navbar-sections.yaml`, `navbar-menu-items.yaml`, and `clock-header.yaml`.
- `views-home/` — the per-device "home" views (`view-home-*.yaml`).
- `views/` — every other view (welcome, cameras, climate, settings, …).

Each view pulls the shared navbar with `!include ../shared/navbar.yaml`. Because YAML anchors
cannot cross files, `navbar.yaml` is the single source for the bar; each view includes its own
copy (the bar is `position: fixed`, so it reads as one continuous navbar across views).

## Deploy loop

1. Edit a file, commit, and push to `main`.
2. The Git pull add-on pulls into `/config` (hourly, or on demand via
   `hassio.addon_restart` targeting the Git pull add-on).
3. Refresh the dashboard. Roll back with `git revert` + push.
