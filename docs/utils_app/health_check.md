# Health Check Notify Command

## Overview

The `health_check_notify` command is a Django management command that runs health checks and sends notifications to Slack when services fail or recover. It provides automated monitoring for your application's critical components.

## How It Works

1. **Health Check Execution**: Runs health checks for specified plugins (database, cache, etc.)
2. **Status Evaluation**: Determines if services are healthy or failing
3. **Notification Logic**:
   - Sends failure alerts when services go down
   - Sends recovery notifications when services come back online
   - Only notifies on status changes to avoid spam
4. **Persistence**: Stores health check results in the database for tracking

## Configuration

### Required Django Apps

Add the following apps to your `INSTALLED_APPS` in `settings.py`:

```python
INSTALLED_APPS = [
    # ... your other apps
    "tinkun_utils",
    "health_check",
    "health_check.db",
    "health_check.cache",
    "health_check.contrib.migrations",
    "health_check.contrib.celery",
    "health_check.contrib.celery_ping",
]
```

### Slack Configuration

Set the Slack webhook URL in your settings:

```python
# Required for Slack notifications
SLACK_WEBHOOK_URL = "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
```

### Health Check Configuration (Optional)

Configure which health check plugins to run:

```python
HEALTH_CHECK = {
    "SUBSETS": {
        "normal": [
            "DatabaseBackend",
            "MigrationsHealthCheck",
            "Cache backend: default",
            "CeleryHealthCheckCelery",
            "CeleryPingHealthCheck",
        ]
    },
}
```

## Usage

### Basic Usage

Run health checks with default settings:

```bash
python manage.py health_check_notify
```

### Advanced Usage

#### Custom Subset

Run specific health check plugins:

```bash
python manage.py health_check_notify --subset database
```

#### Custom Identifier

Set a custom system identifier for notifications:

```bash
python manage.py health_check_notify --identifier "Production API"
```

### Command Options

- `--subset` or `-s`: Specify which health check plugins to run (default: "normal")
- `--identifier` or `-i`: System identifier for Slack notifications (default: `BASE_URL_BACKEND` setting)

## Notification Behavior

### Failure Notifications

- Sent when any health check fails
- Contains details of all failed services
- Includes the system identifier for easy identification

### Recovery Notifications

- Sent when all services return to healthy state after a previous failure
- Only triggered when transitioning from failed to success state
- Celebrates the recovery with a success message

### No Spam Policy

- Notifications are only sent on status changes
- Repeated failures don't generate multiple alerts
- Database tracks notification history to prevent duplicates

## Scheduling

For automated monitoring, schedule the command using cron or your preferred task scheduler:

```bash
# Run every 5 minutes
*/5 * * * * /path/to/python /path/to/manage.py health_check_notify
```
