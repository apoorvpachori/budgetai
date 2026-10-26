# Architecture

## Status

This document records the architecture selected for the initial implementation. The repository currently contains project guidance and planning documentation only; the desktop shell, interface, API, database, and packaging are not implemented yet.

## Goals and constraints

- A person can install and run the app on their own computer without creating a vendor account.
- The MVP is single-device and local-only; it does not provide a LAN-accessible or hosted service.
- The MVP handles one credit-card account per installation.
- The first account is Chase Sapphire Preferred; the MVP treats posted amounts and budgets as USD.
- Initial reporting starts on January 1, 2026. Statements for any month from then onward can be imported in any order; gaps are allowed and must remain visible.
- Import results are reviewable and correctable before they become trusted data.
- Dashboard totals and assistant answers use the same transaction semantics.
- The project remains approachable for contributors and coding agents.

## Shape: local desktop app with a modular Python backend

Use a desktop shell, a React/Vite interface, and one local Python API process. Tauri is the current candidate, not a final decision. The shell owns startup/shutdown and launches the Python API as a sidecar. The API is private to the app and must not accept connections from other devices. Keep clear backend modules, but do not split the first release into separately deployed services.

```text
Desktop window (candidate: Tauri)
  └── React + TypeScript + Vite interface
          │ local-only REST calls: /api/v1
          ▼
      FastAPI sidecar process
        ├── statement import and normalization
        ├── transaction and category services
        ├── dashboard/analytics queries
        ├── read-only assistant tools
        └── SQLAlchemy persistence ── SQLite in OS per-user application data
```

In development, Vite and FastAPI can run separately, with the desktop shell loading the Vite development server. In a packaged app, the UI is bundled into the desktop application and the Python backend is bundled as a platform-specific sidecar. The sidecar listens on loopback only, uses app-scoped access control, and shuts down with the desktop shell. Store the database in the OS per-user application data directory, separate from replaceable app files. Normal use requires no Docker host or network service.

## Planned repository layout

```text
apps/
  desktop/                     Tauri shell candidate and desktop packaging
  ui/                           React + TypeScript + Vite desktop interface
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
deploy/                        Desktop release and packaging guidance
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

The exact Chase Sapphire Preferred statement format is unresolved. Choose PDF statements, CSV transaction exports, or both based on a representative file before implementing the adapter. Do not add arbitrary PDF/OCR support; target the selected Chase layout and keep the adapter boundary for later formats.

Accept statement periods from January 2026 onward in any order. Track actual statement-period coverage so a gap or partial month is never treated as zero or complete.

## Persistence

Use SQLite for the one-account, personal-installation MVP. Store the database in the OS per-user application data directory, separate from the app bundle. Keep SQLAlchemy models and repositories behind the persistence module and use Alembic migrations from the first schema so upgrades are explicit.

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

## Desktop distribution and operations

The intended first release is a desktop application installed on one computer. The app bundle contains code and UI assets, never the live database or uploaded statements. Store durable data in the OS per-user application data directory. Do not start a LAN service. Tauri is a candidate shell; packaging and supported OSes remain open.

Before calling distribution ready, document installation, supported OSes, app-data location, upgrade/migration behavior, and backup/restore. If the Python API uses localhost HTTP, verify that it cannot bind to a LAN/public interface and that requests are scoped to the desktop app.

## Open decisions

1. Pick the first Chase Sapphire Preferred statement format (PDF, CSV, or both).
2. Confirm the desktop shell (Tauri is the current candidate) and supported operating systems.
3. Select the assistant provider strategy, including whether any local model is supported.
4. Select chart and component libraries after the core screens are defined.

## Initial implementation order

1. Create the runnable desktop shell, React/Vite interface, and local FastAPI development setup, with an OpenAPI contract and a health endpoint.
2. Implement one statement import adapter with preview, normalization, duplicate detection, and confirmation.
3. Add transaction correction and deterministic categories/rules.
4. Build dashboard queries, period comparisons, and transaction drill-down.
5. Add simple monthly category budgets.
6. Add read-only assistant tools on top of the same analytics queries.
7. Package desktop builds with per-user local storage and backup guidance; verify no network listener is exposed.
