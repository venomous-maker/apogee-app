# Changelog

Every release of apogee-app, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

- On apogee-framework 0.3.0, with its new modules.
- Products are searchable with Scout: `/api/search?q=`.
- Sign-in with GitHub, Google and the rest at `/auth/{provider}`.
- The dashboards (Telescope, Pulse, the queue's) for admins outside local
  development; `.env.example` lists the new settings.

## 0.2.2 (2026-11-21)

### Requirements

- Uses apogee-framework v0.2.2.

## 0.2.1 (2026-11-20)

### Requirements

- Uses the 0.2.1 release of apogee-framework.

## 0.2.0 (2026-11-20)

### Requirements

- Needs grav 1.9.0 or later, and uses the 0.2.0 releases of the packages it
  depends on.

## 0.1.1 (2026-11-19)

- Uses the newer releases of apogee-framework, apogee-queue, grav-cache, grav-orbit, grav-db, apogee-console, apogee-scheduler, apogee-mail (faster row loading and query building in grav-orbit and grav-db).

## 0.1.0 (2026-11-19)

The first tagged release. Needs grav 1.7.0 or newer.

A small shop built on [Apogee](https://github.com/venomous-maker/apogee-framework), the web framework for [Grav](https://github.com/venomous-maker/grav). It's an example that uses every module, from routes, form requests and policies to queued jobs, mail, the scheduler and the console. The test suite drives all of it through the app.
