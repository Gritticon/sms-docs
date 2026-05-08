---
title: Internal Employee Setup Guide
type: setup
project: sms-api
last-updated: 2026-03-18
---

# Internal Employee Setup Guide

## Overview

All API endpoints now require authentication. Internal employees can be created directly in the database and are used to:
- Login via Swagger UI
- Test all APIs
- Create new schools (only internal employees can create schools)

## Implementation Summary

### What Was Changed

1. **Created Internal Employee System**
   - New model: `app/models/internal_employee.py`
   - New schema: `app/schemas/internal_employee.py`
   - Database table: `internal_employees`

2. **Updated Authentication**
   - Login now supports `school_id=0` for internal employees
   - Auth middleware handles both internal employees and school staff
   - All endpoints now require authentication (except `/auth/login`)

3. **Secured All Endpoints**
   - `/health` - now requires auth
   - `/` - root endpoint requires auth
   - `/customers/` POST - requires internal employee auth
   - All other endpoints already had auth

4. **Swagger Integration**
   - Added Bearer token authentication support
   - Updated OpenAPI documentation with auth instructions

## Setup Instructions

### Step 1: Create Internal Employee in Database

**Option A: Using Python Script (Recommended)**

```bash
# Activate virtual environment
source env/bin/activate  # On Windows: env\Scripts\activate

# Run the script
python scripts/create_internal_employee.py "Admin User" admin@example.com admin123 +1234567890
```

**Script Usage:**
```bash
python scripts/create_internal_employee.py <name> <email> <password> [phone]
```

**Example:**
```bash
python scripts/create_internal_employee.py "Admin User" admin@example.com admin123 +1234567890
```

The script will:
- Check if employee already exists (by email)
- Hash the password securely
- Create the internal employee in the database
- Display success message with employee ID

**Option B: Manual Database Insert (Not Recommended)**

If you need to create manually, you can use SQLAlchemy ORM in a Python shell:

```python
from app.core.session import SessionLocal
from app.models.internal_employee import InternalEmployee
from app.core.auth import hash_password

db = SessionLocal()
try:
    employee = InternalEmployee(
        name="Admin User",
        email="admin@example.com",
        password_hash=hash_password("admin123"),
        phone="+1234567890",
        is_active=True
    )
    db.add(employee)
    db.commit()
    print(f"Created employee: {employee.id}")
except Exception as e:
    db.rollback()
    print(f"Error: {e}")
finally:
    db.close()
```

**Note**: The project rules require using SQLAlchemy ORM, not raw SQL queries.

### Step 2: Login via Swagger

1. **Start the application:**
   ```bash
   # Make sure virtual environment is activated
   source env/bin/activate  # On Windows: env\Scripts\activate
   
   # Start the server
   uvicorn main:app --reload
   ```

2. **Open Swagger UI:**
   - Navigate to: `http://localhost:8000/docs`
   - Or use ReDoc: `http://localhost:8000/redoc`

3. **Login using the `/auth/login` endpoint:**
   - Click on the endpoint to expand it
   - Click "Try it out"
   - **Request Body:**
     ```json
     {
       "email": "admin@example.com",
       "password": "admin123"
     }
     ```
   - **Note**: This is the internal employee login endpoint. For school staff, use `/auth/login/staff` with `school_id`.
   - Click "Execute"
   - **Response:** Copy the `token` from the response (it's in the `data.token` field)

4. **Authorize in Swagger:**
   - Click the **"Authorize"** button (top right, lock icon 🔒)
   - In the "Value" field, enter: `Bearer <your_token>` (replace `<your_token>` with the actual token)
   - Click **"Authorize"**
   - Click **"Close"**
   - You should see a green checkmark indicating you're authenticated

5. **Test APIs:**
   - All endpoints are now accessible
   - Try creating a school: `/customers/` POST
   - Try accessing other endpoints that require authentication

## Usage Examples

### Onboarding a School (Internal Employee Only)

```json
POST /schools/onboard
Authorization: Bearer <token>

{
  "school_id": 12345,
  "school_name": "Test School",
  "contact_email": "school@example.com",
  "contact_phone": "+1234567890",
  "address": "123 Main St",
  "admin_name": "Admin User",
  "admin_email": "admin@school.com",
  "admin_password": "admin123",
  "admin_phone": "+1234567890"
}
```

**Note**: The `/schools/onboard` endpoint:
- Creates the school in the customers table
- Initializes all dynamic tables for the school
- Creates "Admin Department"
- Creates "Admin" Role
- Creates the admin staff member with the provided credentials

### School Staff Login

After a school is onboarded, school staff can login using the `/auth/login/staff` endpoint:
```json
POST /auth/login/staff

{
  "school_id": 12345,  // The school_id created above
  "email": "staff@school.com",
  "password": "password123"
}
```

**Note**: 
- Use `/auth/login/internal` for internal employees (no school_id needed)
- Use `/auth/login/staff` for school staff (school_id required)

## Security Notes

- **All endpoints require authentication** except `/auth/login/internal` and `/auth/login/staff`
- **Only internal employees** can onboard schools (use `/schools/onboard`)
- **Internal employees** can access all schools
- **School staff** can only access their own school's data
- Tokens expire after the configured period (default: 30 days)

## Troubleshooting

### "Token is required" Error
- Make sure you've logged in and copied the token
- Use format: `Bearer <token>` in the Authorization header

### "Only internal employees can onboard schools" Error
- You must login using `/auth/login/internal` endpoint (not `/auth/login/staff`)
- Regular school staff cannot onboard schools
- Ensure you're using an internal employee token

### "Invalid email or password" Error
- Check that the internal employee exists in the database
- Verify the password hash is correct
- Ensure `is_active = TRUE` in the database

## Files Created/Modified

### New Files
- `app/models/internal_employee.py` - Internal employee database model
- `app/schemas/internal_employee.py` - Pydantic schemas for internal employees
- `app/services/internal_employee_service.py` - Business logic for internal employees
- `scripts/create_internal_employee.py` - Script to create internal employees
- `docs/internal-employee-setup.md` - This documentation file

### Modified Files
- `app/models/__init__.py` - Added InternalEmployee export
- `app/schemas/auth.py` - Updated LoginRequest to allow school_id=0
- `app/services/auth_service.py` - Added internal employee login support
- `app/middleware/auth_middleware.py` - Added internal employee authentication
- `app/api/routes/health.py` - Added auth requirement
- `app/api/routes/school.py` - Added auth and internal employee check for customer creation
- `app/main.py` - Added auth to root endpoint
- `app/core/openapi.py` - Added security scheme for Swagger

## Additional Notes

### Multiple Internal Employees

You can create multiple internal employees with different roles/permissions:

```bash
# Create admin user
python scripts/create_internal_employee.py "Admin User" admin@example.com admin123 +1234567890

# Create another internal employee
python scripts/create_internal_employee.py "Support User" support@example.com support123 +1234567891
```

### Password Security

- Passwords are hashed using SHA-256 before storage
- Never store plain text passwords
- Use strong passwords in production
- Consider implementing password complexity requirements

### Internal Employee vs School Staff

- **Internal Employees** (`school_id=0`): Can create schools, access all APIs
- **School Staff** (`school_id>0`): Belong to a specific school, access school-specific data

### Deactivating Internal Employees

To deactivate an internal employee, update the `is_active` field in the database:

```python
from app.core.session import SessionLocal
from app.models.internal_employee import InternalEmployee

db = SessionLocal()
try:
    employee = db.query(InternalEmployee).filter(
        InternalEmployee.email == "admin@example.com"
    ).first()
    if employee:
        employee.is_active = False
        db.commit()
        print("Employee deactivated")
except Exception as e:
    db.rollback()
    print(f"Error: {e}")
finally:
    db.close()
```

