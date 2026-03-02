# reCAPTCHA Handler Decorator

The reCAPTCHA handler decorator provides automatic validation of Google reCAPTCHA tokens for Django REST Framework views.

## Setup

### 1. Settings Configuration

The following settings are automatically configured with default values by the `load_default_config()` function:

```python
# Enable/disable reCAPTCHA validation (defaults to False)
USE_RECAPTCHA = True

# Your reCAPTCHA secret key from Google (defaults to empty string)
RECAPTCHA_SECRET_KEY = "your-secret-key-here"

# Timeout for reCAPTCHA API requests in seconds (defaults to 20)
RECAPTCHA_TIMEOUT = 20
```

**Note**: The `X-Recaptcha-Token` header is automatically added to `CORS_ALLOW_HEADERS` when using the default configuration loader.

### 2. Configuration Validation

The configuration validator will automatically check that if `USE_RECAPTCHA` is `True`, then `RECAPTCHA_SECRET_KEY` must be set to a non-empty value. If validation fails, an `ImproperlyConfigured` exception will be raised with a descriptive error message.

## Usage

### Basic Usage

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from tinkun_utils.decorators.recaptcha_handler_decorator import recaptcha_handler_decorator

class YourAPIView(APIView):
    @recaptcha_handler_decorator
    def post(self, request, *args, **kwargs):
        # Your view logic here
        return Response({"message": "Success"})
```

### Function-based Views

```python
from django.http import JsonResponse
from tinkun_utils.decorators.recaptcha_handler_decorator import recaptcha_handler_decorator

@recaptcha_handler_decorator
def your_view(request):
    # Your view logic here
    return JsonResponse({"message": "Success"})
```

## How It Works

The decorator:

1. **Checks if reCAPTCHA is enabled**: If `USE_RECAPTCHA` is `False`, the decorator does nothing and passes through to your view.

2. **Extracts the token**: Looks for the reCAPTCHA token in the `X-Recaptcha-Token` header.

3. **Validates the token**: Sends a POST request to Google's reCAPTCHA verification API with your secret key and the token.

4. **Handles errors**: Returns appropriate error responses for various failure scenarios.

5. **Calls your view**: If validation succeeds, calls your original view function.

## Client-Side Integration

Your frontend needs to include the reCAPTCHA token in the request header:

```javascript
// Example with fetch API
fetch("/api/your-endpoint/", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "X-Recaptcha-Token": "your-recaptcha-token-here",
  },
  body: JSON.stringify(data),
});
```

## Error Responses

The decorator returns HTTP 403 Forbidden with specific error messages for various scenarios:

- **Missing token**: `"No recaptcha-token was provided."`
- **Invalid token**: Various error messages based on Google's reCAPTCHA error codes
- **Network errors**: `"Something has gone wrong."` for HTTP errors or timeouts

## Error Codes

The decorator handles these reCAPTCHA error codes:

- `missing-input-secret`: Missing secret key
- `invalid-input-secret`: Invalid secret key
- `missing-input-response`: Missing reCAPTCHA response
- `invalid-input-response`: Invalid reCAPTCHA response
- `bad-request`: Malformed request
- `timeout-or-duplicate`: Token timeout or duplicate submission

## Testing

When testing your views, you can disable reCAPTCHA validation:

```python
from django.test import TestCase, override_settings

@override_settings(USE_RECAPTCHA=False)
class YourViewTest(TestCase):
    def test_your_view(self):
        # Your test logic here
        pass
```

Or mock the reCAPTCHA validation:

```python
from unittest.mock import patch

@patch('requests.post')
def test_with_valid_recaptcha(self, mock_post):
    mock_response = MagicMock()
    mock_response.json.return_value = {"success": True}
    mock_post.return_value = mock_response

    # Your test logic here
```

## Security Notes

- Always use HTTPS in production to protect the reCAPTCHA token in transit
- The decorator includes a configurable timeout for reCAPTCHA API calls (default: 20 seconds)
- All validation errors are logged for monitoring purposes
