# Changelog

## 2.0.0 — 2026-11

### BREAKING
- Variables renamed from `ngx_*` / `__ngx_*` to `nginx_*` (see `defaults/main.yml`).
  `ngx_packages` → `nginx_version` + `nginx_extra_packages`, `ngx_sites_*` → `nginx_sites_*`.

### Fixed
- The nginx.org repository was never added (the task was missing), so nginx was installed from the Ubuntu archive
  (e.g. 1.24 instead of 1.30).
- `apt_key` (deprecated) and a `signed-by` pointing at a URL.
- The handler referenced the undefined `nginx_config_start`. The role has no handlers now.

### Added
- deb822 source, apt pin for nginx.org, exact version and hold, a guard against implicit upgrades or replacing
  Ubuntu's nginx.
- Molecule tests (24.04, 26.04, takeover of Ubuntu nginx).
