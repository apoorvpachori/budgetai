# Deployment configuration

- Optimize the first self-hosted deployment for a single installation and a persistent data volume.
- Keep the default deployment understandable and low-dependency: one app service and SQLite for the MVP.
- Never bake database files, uploads, secrets, or user data into an image.
- Document the data mount, upgrade/migration behavior, backup and restore, port binding, and authentication expectations.
- Default to a safe local binding where practical; explain what changes when a user exposes the app to a LAN or the internet.
- Do not add cloud-only dependencies or telemetry to the required path.
