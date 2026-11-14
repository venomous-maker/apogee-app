# apogee-app

A small shop built on [Apogee](https://github.com/venomous-maker/apogee-framework),
the web framework for [Grav](https://github.com/venomous-maker/grav). It's an
example that uses every module, from routes, form requests and policies to
queued jobs, mail, the scheduler and the console. The test suite drives all
of it through the app.

```bash
grav install
cp .env.example .env
grav build
./src/main key:generate
./src/main migrate --seed
./src/main serve                      # http://localhost:8000 — ada@shop.test / password
```

The other commands:

```bash
./src/main queue:work --queue=mail,orders,default,imports
./src/main schedule:work
./src/main shop:import tests/fixtures/products.csv
./src/main shop:report --days=7
./src/main list
```

## What is where

| | |
|---|---|
| `bootstrap/app.grav` | the app: `Application.configure(".")` with its providers, routes, middleware, exception rendering, schedule, commands, migrations and seeders |
| `app/models.grav` | Orbit models: `User` (Authenticatable, Notifiable), `Product` (bound by slug), `Order`, `OrderItem` |
| `app/http.grav` | controllers, form requests, an API resource, a middleware, the product policy |
| `app/domain.grav` | events, listeners (one queued), a subscriber, jobs, mailables, a notification |
| `app/console.grav` | the shop's commands and its service provider |
| `routes/routes.grav` | web routes (session, CSRF) and API routes (`/api`, tokens, throttled) |
| `database/database.grav` | migrations, factories, seeders |
| `resources/views` | Blade-style templates: a layout, pages, forms, components, mail |
| `config/*.json`, `.env` | configuration (`.env.testing` for the tests) |
| `tests/app.grav` | the whole app, end to end |

## Every module, in use

| Module | Where the shop uses it |
|---|---|
| **support** | `Moment` for order dates, "added 3 days ago" and the report's range. `Number.currency` for prices. `Text.slug`, `limit` and `random`. `Collection` in the API's filters and stats and in the report. |
| **container** | Controllers, jobs, listeners, commands and provider `boot()` methods get their services injected. `TimeRequests` is a singleton. |
| **config** | `.env` / `.env.testing` and `config/app.json` / `auth.json`. |
| **events** | `OrderPlaced` reaches a queued listener (the confirmation mail) and a synchronous one (stock). `ShopActivity` subscribes with typed and wildcard (`app.*`) listeners. |
| **http** | Requests and responses, `ProductResource` with conditional fields, paginated collections. |
| **routing** | Controller routes and a resource. Bindings are implicit (`resolveRouteBinding`, by slug) and explicit (`bind("user")`). Also named routes, a signed unsubscribe URL, route groups with middleware, the API prefix, throttling and a 404 fallback. |
| **validation** | Form requests: `RegisterRequest` (with `unique`, `confirmed`, `Password` and `prepareForValidation`), `StoreProductRequest` (with `authorize`), and `PlaceOrderRequest` (with wildcard rules, `exists`, and an `after` hook that checks stock). Also `req.validate` and `ValidationException.withMessages`. |
| **session** | Flashed status messages, old input and errors back on forms, CSRF on every form, the session id renewed at sign-in. |
| **view** | A layout with sections, `@parent`, stacks and includes. Components with slots (`<x-product-card>`, `<x-alert>`). `@csrf`, `@error`, `old()` and `session()`. `@auth` / `@guest`. `@forelse` with `loop`. A view composer and a custom `money()` helper. |
| **auth** | Session sign-in with remember-me, throttled with the cache's rate limiter. API tokens for `/api`. `ProductPolicy` with `before` and `@Guests`, a decision of 404, a gate ability (`view-reports`), and `can:` middleware. |
| **queue** | A job chain (`GenerateInvoice` then `NotifyWarehouse`), a batch (`shop:import`, with `then` and `allowFailures`), a unique job (`RebuildSearchIndex`). Job middleware: rate limited, no overlapping, skip if the batch is cancelled. Models carried by id, retries and backoff, and the database or Redis drivers. |
| **mail** | `OrderConfirmation` in Markdown (a table, a button, a panel), `WelcomeMail` from views (HTML and text), queued mail, and the array mailer in tests. |
| **notifications** | `OrderShipped` over mail and the database inbox (`/api/notifications`). |
| **scheduler** | A daily job on weekdays, an hourly job `onOneServer`, a weekly command with `withoutOverlapping` and its output appended to a log, a five-minute heartbeat `between` hours, a production-only cleanup. |
| **console** | `shop:import` (a dry-run table, then batch jobs), `shop:report` (a box table, `--json`), `shop:admin` (a confirm question, `isolated`), and a closure command (`shop:hello`). |
| **framework** | Providers, maintenance mode (`down --secret`), migrations and seeders, `make:*`, `about`, `route:list`, and `schedule:list`. |
| **grav-orbit / db / cache / redis / logger** | Models, migrations, factories and seeders. The cache stores the front page's products, invoices and the search index. Locks and the rate limiter. Logging channels. |

## Tests

```bash
grav test tests/app.grav
```

These are seven end-to-end tests: the console, the pages, accounts, orders,
authorization, the API, and operations. A small `Browser` keeps cookies and
sends the CSRF token of the last page's form, so the forms are tested the way
a browser submits them.

## Speed

On one machine, with 16 concurrent clients and SQLite:

| Page | Requests per second |
|---|---|
| A product page, rendered from templates | about 2,200 |
| The home page | about 840 |

Each request on the home page is a new visitor, so it writes a new session
file. Each server thread keeps its own database connection.
