# Changelog — Utils App

All notable changes to `tinkun_utils_app` are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [3.4.2] — 2026-02-01

### Changed

- Replaced `django-structlog` dependency with pure `structlog`
- Added `set_message` processor for New Relic log line titles

### Added

- API protection configuration and middleware integration

---

## [3.2.0] — 2026-01-19

### Added

- Structured logging via `structlog`
- Automatic request correlation with request IDs
- Environment-aware logging (colored console in dev, JSON in production)
- User context auto-binding (DRF-compatible — bound after authentication)
- Request timing metrics
- NewRelic-compatible JSON output

---

## [3.1.0]

### Added

- Previous features and improvements
