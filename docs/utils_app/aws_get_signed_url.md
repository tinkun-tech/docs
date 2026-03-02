# AWS S3 GetPresignedUrl Service

## Overview

This feature brings a simple interface for the utils app to generate a pre-signed url from AWS.

### Main Features:

- ✅ **Lazy Initialization**: The S3 Client creates just when it's needed.
- ✅ **Client cache**: Efficient reuse of the S3 Client.
- ✅ **Robust error handling**: Catches and handles AWS exceptions.
- ✅ **Automatic Validation**: Verifies configuration before using the service.
- ✅ **Presigned URLs**: Secure generation of URLs for uploads.
- ✅ **Conditional Configuration**: Can be completely disabled.

## How It Works

### Config values required

Add the following variables in your `settings.py` file of your Django app:

```python
# AWS S3 Configuration
USE_AWS_BUCKET = True  # Enable/Disable service
AWS_STORAGE_BUCKET_NAME = "mi-bucket-s3"
AWS_S3_ACCESS_KEY_ID = "AKIAIOSFODNN7EXAMPLE"
AWS_S3_SECRET_ACCESS_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
AWS_S3_REGION_NAME = "us-east-1"
```

For a better security use enviroment variables:

```bash
# .env file
USE_AWS_BUCKET=true
AWS_STORAGE_BUCKET_NAME=mi-bucket-s3
AWS_S3_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_S3_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
AWS_S3_REGION_NAME=us-east-1
```

```python
# settings.py
import environ

env = environ.Env()
environ.Env.read_env()

USE_AWS_BUCKET = env.bool('USE_AWS_BUCKET', 'false')
AWS_STORAGE_BUCKET_NAME = env.str('AWS_STORAGE_BUCKET_NAME', '')
AWS_S3_ACCESS_KEY_ID = env.str('AWS_S3_ACCESS_KEY_ID', '')
AWS_S3_SECRET_ACCESS_KEY = env.str('AWS_S3_SECRET_ACCESS_KEY', '')
AWS_S3_REGION_NAME = env.str('AWS_S3_REGION_NAME', '')
```

### Basic Import

```python
from tinkun_utils.services import GetPreSignedUrlService

# Instance the service
presigned_url_service = GetPreSignedUrlService()
```

### Generate presigned URL for upload a file

```python
try:
    # Presigned URL Generation
    presigned_data = presigned_url_service.generate_presigned_url("mi-archivo.jpg")

    # The response contains:
    # {
    #     "url": "https://mi-bucket-s3.s3.amazonaws.com/",
    #     "fields": {
    #         "acl": "public-read",
    #         "key": "media/mi-archivo.jpg",
    #         "AWSAccessKeyId": "AKIA...",
    #         "policy": "eyJ...",
    #         "signature": "abc123..."
    #     }
    # }

except ValidationError as e:
    # Handling validation errors
    print(f"Error: {e.detail}")
```

### Monitoring Config:

```python
# settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': 'aws_s3.log',
        },
    },
    'loggers': {
        'tinkun_utils.services.get_signed_url_service': {
            'handlers': ['file'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}
```

---
