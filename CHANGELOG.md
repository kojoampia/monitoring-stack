# Changelog

## 2026-03-31

### Grafana
- **Config:** Added default admin credentials to `.env` (`GF_ADMIN_USER`, `GF_ADMIN_PASSWORD`).
- **Config:** Wired `GF_SECURITY_ADMIN_USER` and `GF_SECURITY_ADMIN_PASSWORD` environment variables into the `grafana` service in `docker-compose.yml`, replacing the default `admin`/`admin` login.
