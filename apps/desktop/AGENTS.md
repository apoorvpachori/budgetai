# Desktop application shell

- The product is a local, single-user desktop app. The shell owns app startup/shutdown, the local backend process, file-open integration if added, and OS packaging.
- The first release targets macOS; keep platform-specific code isolated so other OS targets can be added later.
- Tauri is the current candidate shell, not a final decision. Keep shell-specific code isolated from the React UI and Python product logic.
- The app must not expose its API to other devices on the network. If the backend uses HTTP, bind to loopback only and use a per-launch secret or similarly scoped access control.
- Store financial data in the OS per-user application data directory, not beside the executable or in the repository.
- Keep platform-specific packaging code and configuration here. Document the supported macOS version range and build prerequisites. Tauri supports web frontends and can bundle Python sidecars, but validate build/signing needs before committing to it.
