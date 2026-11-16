# trivy-server

Runs `trivy server` with a persistent cache so the vulnerability DB is downloaded
once and shared by all clients.

Clients (CI jobs) connect with:

```bash
trivy image --server http://trivy-server.<ns>.svc.cluster.local:4954 [--token $TOKEN] IMAGE
```

Notes:
- Single replica (StatefulSet + PVC). The server refreshes the DB on its own.
- `--server` clients skip DB download; use `--scanners vuln` for vulnerability-only scans.
- Set `trivy.dbRepository` / `trivy.javaDbRepository` to an internal mirror if ghcr.io is not reachable.
