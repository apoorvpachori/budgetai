# Desktop packaging and release configuration

- Package the app as a single-user desktop application; do not require a Docker host or network service for normal use.
- Keep the SQLite database and user settings in the OS per-user application data directory, not in the application bundle.
- Never bundle real statements, transaction data, secrets, or user database files in release artifacts.
- Document supported operating systems, installation, upgrades, backup/restore, and app-data location.
- Ensure the Python sidecar or local API binds only to loopback/private local transport and is stopped when the desktop app exits.
- Do not add cloud-only dependencies or telemetry to the required path.
