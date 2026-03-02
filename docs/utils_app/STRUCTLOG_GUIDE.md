# Structured Logging Guide (structlog)

## Overview

Starting with `tinkun_utils_app` v3.3.0, structured logging uses pure `structlog` (replacing `django-structlog`). This implementation correctly binds DRF authenticated users by binding user context AFTER DRF authentication completes in the view layer.

## Features

- **Request Correlation**: Automatic request ID generation and propagation
- **DRF-Compatible User Binding**: User context bound AFTER authentication (works with DRF)
- **Environment-Aware**: Colored console output in development, JSON in production
- **Client IP Tracking**: Automatic extraction of client IP (with proxy support)
- **Exception Tracking**: Full stack traces in structured format
- **Streaming Response Support**: Context preserved during streaming responses
- **NewRelic Compatible**: JSON logs ready for monitoring tool ingestion

## Quick Start

### Enable Structured Logging

Structured logging is automatically enabled when you call `load_default_config()`:

```python
# backend/settings/base.py
from tinkun_utils.app_config.default_config_loader import load_default_config

# This enables structlog + middleware automatically
load_default_config()
```

That's it! No additional configuration required.

### Using Structured Logging

#### Option 1: New Code (Recommended)

Use `get_logger()` for structured logging:

```python
from tinkun_utils.functions import get_logger

logger = get_logger(__name__)

# Log with structured context
logger.info("user_registered", user_id=123, email="user@example.com")
logger.warning("payment_failed", amount=49.99, reason="insufficient_funds")
logger.error("database_error", table="users", operation="insert")
```

#### Option 2: Existing Code (Backward Compatible)

Existing `logging.getLogger()` calls continue working:

```python
import logging

logger = logging.getLogger(__name__)

# Standard logging still works
logger.info("User registered")
logger.warning("Payment failed")
```

## Configuration

### Development Mode (DEBUG=True)

Colored console output for easy reading:

```
[info     ] user_registered                user_id=123 email=user@example.com request_id=abc-123 ip=127.0.0.1
[warning  ] payment_failed                 amount=49.99 reason=insufficient_funds request_id=abc-123 ip=127.0.0.1
```

### Production Mode (DEBUG=False)

JSON output for machine parsing:

```json
{
  "event": "user_registered",
  "user_id": 123,
  "email": "user@example.com",
  "request_id": "abc-123",
  "ip": "192.168.1.100",
  "timestamp": "2026-01-19T19:00:00.000000Z",
  "level": "info",
  "logger": "myapp.views"
}
```

### Force JSON Output (for New Relic, etc.)

If you need JSON logs in a `DEBUG=True` environment (e.g., for New Relic or other log aggregators), set:

```python
# settings.py
STRUCTLOG_USE_JSON = True  # Force JSON output regardless of DEBUG
```

This is useful when:

- You have `DEBUG=True` in staging but still need machine-parseable logs
- Your log aggregator (New Relic, Datadog, etc.) requires JSON format
- You want consistent JSON logs across all environments

## Request Context

The `StructlogMiddleware` automatically binds the following context to all logs during a request:

- `request_id`: Unique UUID for the request (or from `X-Request-ID` header)
- `user_id`: ID of authenticated user (bound AFTER DRF auth completes)
- `ip`: Client IP address (handles X-Forwarded-For for proxied requests)
- `method`: HTTP method (GET, POST, etc.)
- `path`: Request path

Example log output:

```json
{
  "event": "order_created",
  "order_id": 456,
  "request_id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "123",
  "ip": "192.168.1.100",
  "method": "POST",
  "path": "/api/orders/",
  "timestamp": "2026-01-19T19:00:00.123456Z",
  "level": "info"
}
```

### DRF Authentication Note

The middleware binds `user_id` AFTER `get_response()` returns, ensuring DRF token-based authentication has completed. This solves the common issue where user context shows as unauthenticated for DRF endpoints.

## Usage Examples

### View Logging

```python
from django.http import JsonResponse
from tinkun_utils.functions import get_logger

logger = get_logger(__name__)

def create_order(request):
    logger.info("order_creation_started", cart_items=len(request.data.get("items", [])))

    try:
        order = Order.objects.create(user=request.user, ...)
        logger.info("order_created_successfully", order_id=order.id, total=order.total)
        return JsonResponse({"order_id": order.id})
    except Exception as e:
        logger.error("order_creation_failed", error=str(e), user_id=request.user.id)
        raise
```

### Service Logging

```python
from tinkun_utils.functions import get_logger

logger = get_logger(__name__)

class PaymentService:
    def process_payment(self, amount, payment_method):
        logger.info("payment_processing_started", amount=amount, method=payment_method)

        try:
            result = self.charge_card(amount, payment_method)
            logger.info(
                "payment_processed_successfully",
                transaction_id=result.id,
                amount=amount,
                status=result.status
            )
            return result
        except PaymentError as e:
            logger.error(
                "payment_processing_failed",
                amount=amount,
                error_code=e.code,
                error_message=str(e)
            )
            raise
```

### Celery Task Logging

```python
from celery import shared_task
from tinkun_utils.functions import get_logger

logger = get_logger(__name__)

@shared_task
def send_email_task(user_id, template):
    logger.info("email_task_started", user_id=user_id, template=template)

    try:
        send_email(user_id, template)
        logger.info("email_sent_successfully", user_id=user_id, template=template)
    except Exception as e:
        logger.error("email_send_failed", user_id=user_id, error=str(e))
        raise
```

## NewRelic Integration

Structured logs are NewRelic-compatible out of the box. In production (DEBUG=False), logs are output as JSON and can be ingested directly by NewRelic for:

- Log correlation with APM traces
- Custom queries and dashboards
- Alerting based on log attributes
- Distributed tracing

Example NewRelic query:

```sql
SELECT * FROM Log
WHERE event = 'payment_failed'
AND user_id IS NOT NULL
SINCE 1 hour ago
```

## Custom Configuration

### Override Default Configuration

If you need custom logging configuration, set `LOGGING` in your settings **before** calling `load_default_config()`:

```python
# backend/settings/base.py

# Custom logging configuration
LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    # ... your custom config
}

# This will respect your custom LOGGING config
load_default_config()
```

### Custom Request ID Header

By default, the middleware looks for `X-Request-ID` header. You can customize this:

```python
# settings.py
STRUCTLOG_REQUEST_ID_HEADER = "HTTP_X_CORRELATION_ID"
```

### Disable Structured Logging

To disable structured logging entirely:

```python
# Don't call load_default_config(), or set LOGGING before calling it
LOGGING = {
    # Your traditional logging config
}
```

## Troubleshooting

### Issue: Logs not showing structured format

**Solution**: Ensure `load_default_config()` is called in your settings.

### Issue: Request context not appearing in logs

**Solution**: Verify `StructlogMiddleware` is in your `MIDDLEWARE` list. It should be automatically added by `load_default_config()`.

### Issue: Middleware conflicts

**Solution**: The middleware is prepended to the beginning of the middleware chain. If you experience conflicts, you can manually adjust the middleware order after calling `load_default_config()`.

### Issue: Colors not showing in development

**Solution**: Ensure `DEBUG=True` in your settings. The development configuration uses colored console output.

### Issue: user_id not appearing for DRF endpoints

**Solution**: This should now work correctly. The middleware binds user context AFTER `get_response()` returns, which is after DRF authentication has completed.

## Migration Guide

### Migrating from django-structlog

If upgrading from a version using `django-structlog`:

1. **Dependencies**: `django-structlog` is replaced by `structlog` directly
2. **Context fields changed**:
   - Removed: `username`, `is_authenticated`, `duration_ms`
   - Added: `ip`, `method`, `path`
   - `user_id` now bound AFTER auth (fixes DRF compatibility)
3. **No code changes required**: The `get_logger()` function works the same way

### Migrating Existing Code

1. **No immediate changes required**: Existing `logging.getLogger()` calls continue working.

2. **Gradual adoption**: New code can use `get_logger()` for structured logging:

```python
# Old
import logging
logger = logging.getLogger(__name__)
logger.info("User registered")

# New
from tinkun_utils.functions import get_logger
logger = get_logger(__name__)
logger.info("user_registered", user_id=123, email="user@example.com")
```

3. **Update log messages**: Prefer structured context over string formatting:

```python
# Less optimal
logger.info(f"User {user_id} registered with email {email}")

# Better
logger.info("user_registered", user_id=user_id, email=email)
```

## Best Practices

1. **Use structured context**: Pass data as keyword arguments, not in the message string.

2. **Use descriptive event names**: Use snake_case event names that describe what happened.

3. **Log business events**: Log meaningful business events, not just technical details.

4. **Include correlation IDs**: When calling external services, include correlation IDs in logs.

5. **Avoid PII in logs**: Be mindful of logging personally identifiable information.

6. **Use appropriate log levels**:
   - `DEBUG`: Detailed diagnostic information
   - `INFO`: General informational messages
   - `WARNING`: Warning messages for potentially harmful situations
   - `ERROR`: Error messages for serious problems
   - `CRITICAL`: Critical messages for very severe issues

## Performance Considerations

- Structured logging has minimal overhead
- JSON serialization happens only in production
- Development console rendering is optimized for readability
- Context binding uses `contextvars` (thread-safe and async-friendly)

## Additional Resources

- [structlog documentation](https://www.structlog.org/)
- [NewRelic Logs documentation](https://docs.newrelic.com/docs/logs/)

## Support

For issues or questions:

- File an issue in the tinkun_utils_app repository
- Contact the Tinkun infrastructure team
