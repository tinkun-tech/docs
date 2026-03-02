# User Status Management System

## Overview

The User Status Management System provides functionality to suspend and reactivate users in Django applications. It includes a service layer, Django admin interface, and templates for bulk user operations via email lists.

## How It Works

1. **Service Layer**: `UserStatusService` handles the core logic for user suspension/reactivation
2. **Admin Interface**: `UserAdminMixin` provides Django admin pages for bulk operations
3. **Missing User Handling**: Reports which emails don't correspond to existing users
4. **Validation**: Comprehensive error handling and validation

## Configuration

### Required Django Apps

Add the following app to your `INSTALLED_APPS` in `settings.py`:

```python
INSTALLED_APPS = [
    # ... your other apps
    "tinkun_utils",
]
```

### User Service Configuration

**REQUIRED**: You must implement and configure a user service that handles the actual suspension/reactivation logic:

```python
# Required setting - path to your user service implementation
USER_SERVICE = "myapp.services.MyUserService"
```

### User Service Implementation

Your user service must implement the `UserServiceInterface`:

```python
from typing import List

from django.contrib.auth.models import AbstractUser
from django.db.models import QuerySet

from tinkun_utils.common.interfaces import UserServiceInterface


class MyUserService(UserServiceInterface):
    def suspend_users(self, users: QuerySet[AbstractUser]) -> None:
        """Suspend the given list of users"""
        # Your suspension logic here
        users.update(is_active=False)

    def activate_users(self, users: QuerySet[AbstractUser]) -> None:
        """Activate/reactivate the given list of users"""
        # Your activation logic here
        users.update(is_active=True)
```

### Using the Django Admin Interface

#### 1. Setup Admin Class

Inherit from `UserAdminMixin` in your user admin:

```python
from django.contrib import adminl

from tinkun_utils.admin import UserAdminMixin

from backend.models import User

@admin.register(User)
class UserAdmin(UserAdminMixin):
    # Your existing admin configuration
    list_display = ("email", "first_name", "last_name")
    list_filter = ("is_staff", "is_superuser", "groups", "status")

    # UserAdminMixin adds suspend/reactivate functionality automatically
```

#### 2. Admin Interface Features

The admin interface provides:

- **Suspend Users**: Bulk suspend users by email list
- **Reactivate Users**: Bulk reactivate users by email list
- **Email Validation**: Validates email format and checks for empty lists
- **Missing User Reports**: Shows which emails don't exist in the database
- **Error Handling**: Displays validation errors and service errors

#### 3. Admin Workflow

1. Navigate to your User admin in Django admin
2. Click "Suspend Users" or "Reactivate Users" button
3. Enter email addresses (one per line) in the textarea
4. Click submit to process
5. View confirmation page with results

### Email List Format

user1@example.com
user2@example.com
user3@example.com

## Template Customization

### Available Templates

You can override these templates in your project:

tinkun_utils/admin/user/
├── change_list.html # Admin list view with action buttons
├── suspend_users.html # Suspend users form
├── suspend_confirmation.html # Suspend results page
├── reactivate_users.html # Reactivate users form
└── reactivate_confirmation.html # Reactivate results page

## Error Handling

### Admin Level Errors

The admin interface displays user-friendly error messages for:

- Empty email lists
- Invalid email formats
- Service validation errors
- Missing user accounts
