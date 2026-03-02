# Changelog

All notable changes to `tinkun_utils_app` are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [v4.0.0] — 2026-02-26

### Changed

- Bump Python version to 3.14.2 and fix tests

---

## [v3.4.3] — 2026-02-18

### Added

- Migrate logging to use a custom `get_logger` function and structured event messages
- Add new final option to Salesforce API

### Changed

- Integrate `django-structlog` for structured logging

---

## [v3.4.2] — 2026-02-09

### Changed

- Replaced `django-structlog` dependency with pure `structlog`
- Added `set_message` processor for New Relic log line titles

---

## [v3.4.1] — 2026-02-09

### Fixed

- Patch release for v3.4.0

---

## [v3.4.0] — 2026-01-29

### Added

- API protection configuration and middleware integration

---

## [v3.3.2] — 2026-01-21

### Fixed

- Fix New Relic structured logging integration

---

## [v3.3.1] — 2026-01-20

### Changed

- Refactor logger

---

## [v3.3.0] — 2026-01-20

### Added

- Structured logging integration

---

## [v3.2.1] — 2025-07-31

### Fixed

- Rename `USE_AWS` env values for settings

---

## [v3.2.0] — 2025-07-29

### Added

- AWS S3 `GetPresignedUrlService` with default configuration and S3 client

---

## [v3.1.0] — 2025-07-28

### Added

- Google reCAPTCHA handler decorator and required configs

---

## [v3.0.1] — 2025-06-30

### Fixed

- Fix data wrapper in Salesforce utility

---

## [v3.0.0] — 2025-06-25

### Added

- Add `test_app` (excluded from packaging)

### Removed

- Remove testing models

---

## [v2.7.1] — 2025-06-25

### Added

- Salesforce: new `api_type` param on `call_salesforce_api` to change the POST target URL

---

## [v2.7.0] — 2025-06-23

### Added

- Blocked Domain validation against external API
- New Relic integration
- Salesforce integration for sending user data to external API

---

## [v2.6.3] — 2025-05-13

### Fixed

- Handle Turnstile exceptions correctly

---

## [v2.6.2] — 2025-03-05

### Fixed

- Remove `self` param from `turnstile_handler_decorator` for compatibility with auth app

---

## [v2.6.1] — 2025-01-16

### Fixed

- Fix `@` operator usage from v2.6.0

---

## [v2.6.0] — 2025-01-03

### Added

- Retry decorator with exponential backoff (similar to Celery retry)
  - `autoretry_for`, `max_retries`, `retry_backoff`, `retry_jitter` arguments

---

## [v2.5.0] — 2024-12-27

### Added

- Custom permission class `RequiresProgramActive`

---

## [v2.4.1] — 2024-12-20

### Fixed

- Rename initial migration `0001`

---

## [v2.4.0] — 2024-12-12

### Added

- `BlockedDomain` model with email validation against blocked domains

---

## [v2.3.2] — 2024-11-29

### Fixed

- Fix `get_content_type` in API content retrieval

---

## [v2.3.1] — 2024-11-28

### Fixed

- Fix 204 responses — `create_response` returns nothing when data is `None`

### Added

- Header support for `get_content_from_api` and `post_content_to_api`
- New API utils: `put_content_to_api_with_authentication`, `delete_content_from_api_with_authentication`

---

## [v2.3.0] — 2024-10-11

### Changed

- `update_dictionary_setting_values` now initializes dict if `None` and only updates missing keys

---

## [v2.2.0] — 2024-06-19

### Added

- `TestCaseExtended` for common test patterns

---

## [v2.1.0] — 2024-06-19

### Added

- Turnstile decorator

---

## [v2.0.0] — 2024-05-28

### Added

- Official release

---

## [v0.0.2] — 2024-04-03

### Changed

- Improve package metadata and release action

---

## [v0.0.1] — 2024-04-03

### Added

- Initial release
