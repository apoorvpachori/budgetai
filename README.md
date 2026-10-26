# BudgetAI

BudgetAI is a self-hosted budgeting app built around a simple workflow: import a credit-card statement, review and categorize its transactions, explore spending, and ask questions about the data in natural language.

The goal is a private, understandable alternative to services such as Mint or Rocket Money. Each installation belongs to the person or household running it; the project should not require a BudgetAI account or a BudgetAI-hosted backend.

## Project status

This repository is at the planning and foundation stage. The application itself has not been implemented yet. The architecture and contribution instructions below are the current project decisions, not a claim that those features or commands already work. The detailed MVP requirements are in [`docs/prd.md`](docs/prd.md).

## MVP

The first useful release is deliberately narrow:

1. Import a statement for one credit-card account per installation.
2. Preview parsed transactions and catch duplicates before saving.
3. Categorize transactions, with a way for the user to correct categories and add simple rules.
4. Show spending by period, category, and merchant; let the owner set simple monthly category budgets.
5. Answer natural-language questions using the installation's transaction history and show the date range and transactions or totals behind each answer.

The initial import covers roughly two recent statement periods and can grow as the owner adds later statements. The statement issuer and file format are not selected yet. The first importer should target one representative format and use an adapter boundary so additional formats can be added later. Bank connections, multi-card workflows, shared hosted accounts, and financial advice are outside the first MVP. See the PRD for release boundaries, user journeys, requirements, and open decisions.

## Architecture direction

- **Monorepo:** `apps/web` contains the browser UI; `apps/api` contains the backend; `deploy` contains self-hosting configuration; `docs` contains design and operating guidance.
- **Frontend:** React with TypeScript and Vite. This is a client-rendered app; the Python API owns the product logic and data. Vite provides the development server and static production build.
- **Backend:** Python with FastAPI. Use a modular monolith, with HTTP routes kept thin and product logic in application/domain services.
- **API contract:** REST endpoints under `/api/v1`. FastAPI's OpenAPI schema is the source for frontend API types.
- **Persistence:** SQLite for the one-account, single-installation MVP, stored in a persistent mounted data directory. Use SQLAlchemy for database access and Alembic for schema migrations. Keep database access behind a small persistence layer; only consider a second database when an actual deployment need calls for it.
- **Self-hosting:** Aim for one Docker Compose service and one persistent data volume for the first release. Build the Vite frontend into static assets and serve them from the same origin as the API. Do not require a separate database container or external account for a basic installation.
- **Imports:** Separate file-specific parsing from transaction normalization, duplicate detection, preview, and persistence. The user confirms a preview before it changes stored data.
- **Categorization:** Deterministic built-in categories and user-editable rules/overrides come first. Model-generated category suggestions can be optional later.
- **Assistant:** Natural-language requests call a small set of read-only application services for things like spending by category, merchant search, and period comparisons. The assistant must not write transactions or issue arbitrary SQL.

For the full component boundaries, data flow, and open decisions, see [`docs/architecture.md`](docs/architecture.md).
For detailed product scope and acceptance criteria, see [`docs/prd.md`](docs/prd.md).

## Repository map

```text
.
├── AGENTS.md                 # Instructions for coding agents working in this repository
├── README.md                 # Product scope, status, architecture, and contributor entry point
├── apps/
│   ├── AGENTS.md             # Shared rules for applications
│   ├── api/
│   │   └── AGENTS.md         # Python/FastAPI backend rules
│   └── web/
│       └── AGENTS.md         # React/Vite frontend rules
├── deploy/
│   └── AGENTS.md             # Self-hosting and container rules
└── docs/
    ├── AGENTS.md             # Documentation maintenance rules
    ├── architecture.md       # System design and current decisions
    └── prd.md                # MVP product requirements and acceptance criteria
```

Every tracked project directory has a scoped `AGENTS.md`. `.git` and future generated, dependency, cache, build, and runtime-data directories are intentionally excluded.

## Development

There is no runnable application yet, so development and deployment commands will be added when the initial frontend and API are scaffolded. The intended local workflow is to run Vite and FastAPI separately, with Vite proxying `/api` requests to the backend. The intended self-host workflow is Docker Compose with the application data stored in a named or bind-mounted volume.

Do not put real credit-card statements or transaction exports in the repository. Use synthetic or explicitly sanitized sample data for development documentation and examples.

## Privacy and deployment expectations

- The user controls the host, database, and backups.
- Financial data stays on that host unless the user explicitly configures an external model provider.
- External AI requests, when implemented, must be opt-in and documented with what data is sent.
- A basic local installation should not need a cloud account, bank-linking provider, or telemetry service.
- Deployment guidance must explain persistent storage, backup/restore, network binding, and any authentication requirements before recommending exposure beyond the local machine.

## Decisions still to make

- Which issuer and statement format the first importer will support.
- The first-run and authentication experience for a single-user installation, especially when reachable on a home network.
- Which local or external model providers, if any, will be supported by the first assistant implementation.
- The exact charting and UI component libraries.

Keep these decisions visible here until implementation settles them. Avoid introducing dependencies just to make an undecided feature look complete.

## Useful upstream documentation

- [React: Creating a React App](https://react.dev/learn/creating-a-react-app)
- [Vite: Getting Started](https://vite.dev/guide/)
- [FastAPI: Features and automatic API documentation](https://fastapi.tiangolo.com/features/)
- [SQLAlchemy ORM](https://docs.sqlalchemy.org/en/20/orm/)
- [Alembic Tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html)
