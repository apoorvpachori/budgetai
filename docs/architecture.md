# Architecture

## Status

This document records the architecture selected for the initial implementation. The repository currently contains project guidance and planning documentation only; the frontend, API, database, and deployment are not implemented yet.

## Goals and constraints

- A person can run the app on hardware they control without creating a vendor account.
- The MVP handles one credit-card account per installation.
- Import results are reviewable and correctable before they become trusted data.
- Dashboard totals and assistant answers use the same transaction semantics.
- The project remains approachable for contributors and coding agents.

## Shape: modular monolith

Use one Python API process and one browser app. Keep clear internal modules, but do not split the first release into separately deployed services. Statement parsing, deterministic categorization, dashboard queries, and read-only assistant tools are ordinary backend modules.

```text
Browser (React + TypeScript)
        │ same-origin REST calls: /api/v1
        ▼
FastAPI application
  ├── statement import and normalization
  ├── transaction and category services
  ├── dashboard/analytics queries
  └── read-only assistant tools
        │
        ▼
SQLAlchemy persistence layer ── SQLite file on persistent host volume
```

In development, Vite and FastAPI run as separate processes for fast reloads; Vite proxies API requests. In a basic production install, build the web app to static assets and serve it from the same origin as FastAPI. Package both in one Docker image and run one Compose service with a persistent data mount. This avoids CORS setup and a mandatory database service for a personal installation.

## Planned repository layout

```text
apps/
  web/                         React + TypeScript + Vite
  api/                         FastAPI application
    src/budgetai/
      api/                     HTTP routes and request/response schemas
      imports/                 File adapters, normalization, preview, dedupe
      transactions/            Transaction and correction behavior
      categories/              Built-in categories and user rules
      budgets/                 Monthly category limits and progress
      analytics/               Shared spending queries for charts and agent
      assistant/               Provider-neutral, read-only tool orchestration
      persistence/             SQLAlchemy models and repositories
    migrations/                Alembic schema history
deploy/                        Dockerfile and Compose configuration
docs/                           Architecture, operations, and decisions
```

Create nested directories as implementation arrives. Every tracked project directory gets a short, local `AGENTS.md`; `.git`, dependencies, caches, generated output, build output, and runtime data do not.

## API and frontend contract

- Use a REST API with an `/api/v1` prefix.
- Keep the FastAPI-generated OpenAPI document as the API contract.
- Generate TypeScript API types/client code from that schema rather than manually copying request/response definitions.
- Expose aggregate queries through explicit endpoints/services, so the dashboard and assistant agree on date ranges, refunds, payment treatment, and category semantics.

Potential first API slices are statement upload/preview, import confirmation, transaction listing and edits, category/rule management, dashboard summaries, category budgets, and assistant questions. Product behavior and release criteria are defined in [`prd.md`](prd.md); exact route names should be chosen alongside the first vertical slice.

## Import and transaction flow

```text
Upload
  → validate file and size
  → parse with one format-specific adapter
  → normalize rows into a canonical transaction shape
  → compute stable source fingerprint and identify duplicates
  → show preview and validation issues
  → user confirms
  → persist import and transactions
  → apply deterministic categorization rules
```

Parsing must not write to the database. Re-uploading the same statement must not silently duplicate rows. Keep the original transaction description and enough import metadata to troubleshoot parser behavior. Do not retain the uploaded file indefinitely by default; make retention an explicit product decision.

Represent amounts as integer minor units and include ISO currency. Set and document a consistent sign convention for purchases, refunds, and payments. Preserve transaction date separately from import time. Treat statement rows as untrusted input and report rows that cannot be parsed instead of silently dropping them.

The statement format is unresolved. Choose one real, representative export before implementing the adapter; prefer a predictable CSV export if it contains the needed transaction details, and add PDF/OCR only if that source requires it.

## Persistence

Use SQLite for the one-account, personal-installation MVP. Its main benefit is self-hosting simplicity: one persistent file, no database service to configure. Keep SQLAlchemy models and repositories behind the persistence module and use Alembic migrations from the first schema so upgrades are explicit.

Initial conceptual entities:

- `card_accounts`: one row in MVP; keeps the account boundary explicit for future expansion.
- `statement_imports`: format/source metadata, date range, row counts, and import status.
- `transactions`: canonical dates, signed amount in minor units, currency, original description, normalized merchant, category, and source fingerprint.
- `categories` and `categorization_rules`: stable category identity and user-editable matching rules.
- `budgets`: monthly category limits, calculated against posted-date spending.

Do not build multi-account workflows solely because the schema has an account identifier. SQLite-to-PostgreSQL portability is not promised until a second backend is actually needed and exercised.

## Categorization and corrections

Start with deterministic built-in categories and simple user rules. User corrections must override defaults/rules consistently and survive re-imports. Store enough provenance to distinguish an untouched suggestion from a user choice. Model-based suggestions, if added, should remain optional and must never overwrite a user correction silently.

## Natural-language assistant

The assistant consumes the same analytics and transaction services as the UI. Expose a small set of typed tools, for example:

- spending by category or budget progress for an explicit date range;
- spending over time for a category or merchant;
- search transactions with bounded filters;
- compare two explicit periods.

The assistant is read-only in the MVP. Tool inputs need date/range limits and result-size limits. Responses should distinguish calculated facts from interpretation and show the date range plus supporting totals or transactions. Do not let a model run raw SQL or gain write access to the transaction store.

Keep the model provider behind an interface. Any external provider must be explicitly configured by the owner and documentation must describe what transaction data leaves the host. No model-provider choice has been made yet.

## Self-hosting and operations

The intended first release uses one Compose service and one persistent volume. The container image contains application code and static frontend assets, never the live database or uploaded statements. Configuration should use documented environment variables, with safe defaults and no required telemetry.

Before calling deployment ready, document first run, upgrade/migration behavior, backup/restore, network binding, and authentication. The MVP user/account model and the right authentication flow for LAN access are unresolved; do not imply unauthenticated network access is safe.

## Open decisions

1. Pick the initial card issuer and statement file format.
2. Decide whether the first release is local-only by default or has a minimal single-user login for LAN access.
3. Select the assistant provider strategy, including whether any local model is supported.
4. Select chart and component libraries after the core screens are defined.

## Initial implementation order

1. Create the runnable React/Vite and FastAPI development setup, with an OpenAPI contract and a health endpoint.
2. Implement one statement import adapter with preview, normalization, duplicate detection, and confirmation.
3. Add transaction correction and deterministic categories/rules.
4. Build dashboard queries, period comparisons, and transaction drill-down.
5. Add simple monthly category budgets.
6. Add read-only assistant tools on top of the same analytics queries.
7. Package the app for Docker Compose with persistent storage and backup guidance.
