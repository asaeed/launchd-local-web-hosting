---
name: launchd-local-web-hosting
description: Keep a local macOS web development server running across terminal closes, login, and crashes with a per-user launchd agent. Use when setting up or maintaining persistent local web hosting; not for production deployment.
---

# Persistent local web hosting

Use a **per-user LaunchAgent** (`~/Library/LaunchAgents`), never a root LaunchDaemon. Inspect the project's run command, ports, environment file, and any existing agent before writing or replacing one. Do not put secrets into the plist; let the project read its local env file.

Use absolute executable paths (`command -v node`, `command -v npm`) because launchd has a minimal PATH. Set `WorkingDirectory`, `RunAtLoad`, `KeepAlive`, and separate stdout/stderr log files. After installation, use `launchctl bootstrap gui/$(id -u) <plist>`, then `launchctl kickstart -k gui/$(id -u)/<label>`; verify with `launchctl print`, the expected port, and a local HTTP request. Stop or uninstall an existing agent only with the user's approval.

## Hot reload

Keep frontend HMR and backend reload separate:

- Vite (or the frontend dev server) provides browser hot reload.
- For Node backends, prefer scoped watching over broad `node --watch` when file descriptors are limited:

```sh
node --watch-path=server --watch-path=etc/local.env --watch-preserve-output \
  --env-file-if-exists=etc/local.env --import tsx server/index.ts
```

Watch only the project's source/config paths. If a relevant edit does not reload, add that path explicitly; do not broaden the watcher without checking its file-descriptor impact.

For a Node + Vite app, run the scoped backend watcher and Vite together under one launchd-managed parent process. A one-time restart is needed after changing the LaunchAgent command itself.
