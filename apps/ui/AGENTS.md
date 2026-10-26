# Desktop interface

- Frontend implementation is planned in React, TypeScript, and Vite as a client-rendered interface inside a desktop webview. Browser mode is for development only unless product scope changes.
- The interface communicates with the local backend over the versioned REST API. Do not embed database logic, statement parsing, category rules, or spending calculations in UI components.
- Prefer API types generated from the backend OpenAPI schema over maintaining a second handwritten contract.
- Display amounts using the transaction currency and integer minor-unit values from the API; do not calculate money with floating-point arithmetic.
- Make import review, duplicate warnings, loading/error states, and manual corrections clear. Correctness and user control matter more than decorative dashboards.
- Do not send transaction data to analytics, logging, or third-party UI services.
- Do not assume or add access from other devices on the LAN.
- Read the nearest nested `AGENTS.md` before adding a frontend subdirectory.
