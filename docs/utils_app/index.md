# Utils App

A collection of reusable utilities, configurations, and components for Tinkun Django projects.

## Installation

```bash
pip install tinkun_utils_app
```

## Requirements

- Python 3.14.2+
- Django 5.0+

## Features

### Core Utilities

- **Default Configuration Loader** — Centralized configuration management with sensible defaults
- **Response Utilities** — Standardized API response helpers
- **Test Utilities** — Common test helpers and base classes
- **Time Functions** — Timezone-aware datetime utilities
- **Dynamic Import** — Safe dynamic class and model loading

### Security & Validation

- **Turnstile Integration** — Cloudflare Turnstile support for bot protection
- **reCAPTCHA Support** — Google reCAPTCHA v3 integration ([guide](recaptcha_handler_decorator.md))
- **Blocked Domains** — Email domain validation and blocking
- **Program Active Validation** — Time-based program availability checks

### Third-Party Integrations

- **Salesforce** — Lead submission and campaign management
- **WebPurify** — Content moderation service integration
- **AWS S3** — Pre-signed URL generation for secure file uploads ([guide](aws_get_signed_url.md))
- **NewRelic** — Application performance monitoring integration

### Infrastructure

- **Health Checks** — Django health check configuration for monitoring ([guide](health_check.md))
- **JWT Authentication** — Simple JWT configuration for DRF
- **CORS Headers** — Pre-configured CORS settings
- **Structured Logging** — JSON logging with request correlation ([guide](STRUCTLOG_GUIDE.md))
- **User Status Management** — Suspend/reactivate users via admin ([guide](user_status.md))

## Quick Start

```python
# settings.py
INSTALLED_APPS = [
    ...
    "tinkun_utils",
]
```

## Links

- [Changelog](changelog.md)
- [GitHub Releases](https://github.com/tinkun-tech/utils-app/releases)
