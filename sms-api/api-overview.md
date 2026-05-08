---
title: API Overview for UI Integration
type: api-reference
project: sms-api
last-updated: 2026-03-18
---

# API Overview for UI Integration

## Base Configuration

### Base URLs
- **Development**: `http://localhost:8000`
- **Production**: `https://api.gritticon.com`
- **Note**: No base path prefix. Routes sit at the root of `api.gritticon.com`.

### Authentication
All endpoints (except login) require Bearer token authentication:
- **Header**: `Authorization: Bearer <token>`
- Token is obtained from `/auth/login`

---

## 1. Authentication APIs

### 1.1 Internal Employee Login
- **Endpoint**: `POST /auth/login/internal`
- **Auth Required**: No
- **Request Body**:
  ```json
  {
    "email": "string",
    "password": "string"
  }
  ```
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "data": {
      "id": 1,
      "name": "string",
      "email": "string",
      "token": "jwt_token",
      "user_id": 1
    },
    "message": "Login successful"
  }
  ```
- **Note**: Internal employees can access all schools. Use this endpoint for admin users.

### 1.2 School Staff Login
- **Endpoint**: `POST /auth/login/staff`
- **Auth Required**: No
- **Request Body**:
  ```json
  {
    "school_id": 1,
    "email": "string",
    "password": "string"
  }
  ```
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "data": {
      "id": 1,
      "name": "string",
      "email": "string",
      "token": "jwt_token",
      "user_id": 1
    },
    "message": "Login successful"
  }
  ```
- **Note**: School staff can only access their specific school's data.

### 1.3 Logout
- **Endpoint**: `POST /auth/logout`
- **Auth Required**: Yes
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "message": "Logout successful"
  }
  ```

### 1.4 Refresh Token
- **Endpoint**: `POST /auth/refresh`
- **Auth Required**: Yes
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "data": {
      "token": "new_jwt_token"
    },
    "message": "Token refreshed successfully"
  }
  ```

### 1.5 Verify Token
- **Endpoint**: `GET /auth/verify`
- **Auth Required**: Yes
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "data": {},
    "message": "Token is valid"
  }
  ```

---

## 2. Health Check

### 2.1 Health Check
- **Endpoint**: `GET /health`
- **Auth Required**: Yes
- **Response**: `200 OK`
  ```json
  {
    "status": "healthy"
  }
  ```

---

## 3. School APIs (Customers)

### 3.1 Onboard School
- **Endpoint**: `POST /schools/onboard`
- **Auth Required**: Yes (Internal employees only)
- **Request Body**:
  ```json
  {
    "school_id": 1,
    "school_name": "string",
    "contact_email": "string|null",
    "contact_phone": "string|null",
    "address": "string|null",
    "admin_name": "string",
    "admin_email": "string",
    "admin_password": "string",
    "admin_phone": "string|null"
  }
  ```
- **Response**: `201 Created`
  ```json
  {
    "success": true,
    "data": {
      "school_id": 1,
      "school_name": "string",
      "admin_staff_id": 1,
      "admin_department_id": 1,
      "admin_role_id": 1
    },
    "message": "School onboarded successfully"
  }
  ```
- **Note**: This endpoint creates the school, initializes all dynamic tables, and creates the admin staff member.

### 3.2 List Schools
- **Endpoint**: `GET /schools/`
- **Auth Required**: Yes (Internal employees only)
- **Query Parameters**:
  - `page` (optional, default: 1, min: 1)
  - `page_size` (optional, default: 100, min: 1, max: 1000)
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "data": [
      {
        "school_id": 1,
        "school_name": "string",
        "status": "active|inactive|suspended|trial",
        "charged_amount": 0.0,
        "contact_email": "string|null",
        "contact_phone": "string|null",
        "address": "string|null",
        "primary_color": "#7C3AED|null",
        "secondary_color": "#4F46E5|null",
        "kyc_modules_config": "{\"attendance\":true,\"fees\":false}|null",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "message": "Operation successful",
    "total": 100,
    "page": 1,
    "page_size": 100
  }
  ```

### 3.3 Get School
- **Endpoint**: `GET /schools/{school_id}`
- **Auth Required**: Yes (Internal employees only)
- **Path Parameters**: `school_id` (integer)
- **Response**: `200 OK` (same structure as single school object)

### 3.4 Update School
- **Endpoint**: `PUT /schools/{school_id}`
- **Auth Required**: Yes
- **Path Parameters**: `school_id` (integer)
- **Request Body** (all fields optional):
  ```json
  {
    "school_name": "string|null",
    "contact_email": "string|null",
    "contact_phone": "string|null",
    "address": "string|null",
    "status": "active|inactive|suspended|trial|null",
    "charged_amount": 0.0|null
  }
  ```
- **Response**: `200 OK` (single customer object)

---

## 4. Staff APIs

### 4.1 List Staff
- **Endpoint**: `GET /staff/`
- **Auth Required**: Yes
- **Query Parameters**:
  - `page` (optional, default: 1, min: 1)
  - `page_size` (optional, default: 100, min: 1, max: 1000)
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "data": [
      {
        "id": 1,
        "name": "string",
        "email": "string",
        "phone": "string|null",
        "role_id": 1|null,
        "department_id": 1|null,
        "is_active": true,
        "last_login_at": "datetime|null",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "message": "Operation successful"
  }
  ```

### 4.2 Get Staff
- **Endpoint**: `GET /staff/{staff_id}`
- **Auth Required**: Yes
- **Path Parameters**: `staff_id` (integer)
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "data": { /* staff object */ },
    "message": "Operation successful"
  }
  ```

### 4.3 Create Staff
- **Endpoint**: `POST /staff/`
- **Auth Required**: Yes
- **Request Body**:
  ```json
  {
    "name": "string",
    "email": "string",
    "password": "string",
    "phone": "string|null",
    "role_id": 1|null,
    "department_id": 1|null
  }
  ```
- **Response**: `201 Created` (staff detail response)

### 4.4 Update Staff
- **Endpoint**: `PUT /staff/{staff_id}`
- **Auth Required**: Yes
- **Path Parameters**: `staff_id` (integer)
- **Request Body** (all fields optional):
  ```json
  {
    "name": "string|null",
    "email": "string|null",
    "phone": "string|null",
    "role_id": 1|null,
    "department_id": 1|null,
    "is_active": true|null
  }
  ```
- **Response**: `200 OK` (staff detail response)

### 4.5 Delete Staff
- **Endpoint**: `DELETE /staff/{staff_id}`
- **Auth Required**: Yes
- **Path Parameters**: `staff_id` (integer)
- **Response**: `204 No Content`

---

## 5. Class APIs

### 5.1 List Classes
- **Endpoint**: `GET /classes/`
- **Auth Required**: Yes
- **Query Parameters**:
  - `page` (optional, default: 1, min: 1)
  - `page_size` (optional, default: 100, min: 1, max: 1000)
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "data": [
      {
        "id": 1,
        "name": "string",
        "code": "string",
        "description": "string|null",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "message": "Operation successful"
  }
  ```

### 5.2 Get Class
- **Endpoint**: `GET /classes/{class_id}`
- **Auth Required**: Yes
- **Path Parameters**: `class_id` (integer)
- **Response**: `200 OK` (class detail response)

### 5.3 Create Class
- **Endpoint**: `POST /classes/`
- **Auth Required**: Yes
- **Request Body**:
  ```json
  {
    "name": "string",
    "code": "string",
    "description": "string|null"
  }
  ```
- **Response**: `201 Created` (class detail response)

### 5.4 Update Class
- **Endpoint**: `PUT /classes/{class_id}`
- **Auth Required**: Yes
- **Path Parameters**: `class_id` (integer)
- **Request Body** (all fields optional):
  ```json
  {
    "name": "string|null",
    "code": "string|null",
    "description": "string|null"
  }
  ```
- **Response**: `200 OK` (class detail response)

### 5.5 Delete Class
- **Endpoint**: `DELETE /classes/{class_id}`
- **Auth Required**: Yes
- **Path Parameters**: `class_id` (integer)
- **Response**: `204 No Content`

---

## 6. Section APIs

### 6.1 List Sections
- **Endpoint**: `GET /sections/`
- **Auth Required**: Yes
- **Query Parameters**:
  - `class_id` (optional, integer) - Filter by class
  - `page` (optional, default: 1, min: 1)
  - `page_size` (optional, default: 100, min: 1, max: 1000)
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "data": [
      {
        "id": 1,
        "name": "string",
        "code": "string",
        "class_id": 1,
        "description": "string|null",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "message": "Operation successful"
  }
  ```

### 6.2 Get Section
- **Endpoint**: `GET /sections/{section_id}`
- **Auth Required**: Yes
- **Path Parameters**: `section_id` (integer)
- **Response**: `200 OK` (section detail response)

### 6.3 Create Section
- **Endpoint**: `POST /sections/`
- **Auth Required**: Yes
- **Request Body**:
  ```json
  {
    "name": "string",
    "code": "string",
    "class_id": 1,
    "description": "string|null"
  }
  ```
- **Response**: `201 Created` (section detail response)

### 6.4 Update Section
- **Endpoint**: `PUT /sections/{section_id}`
- **Auth Required**: Yes
- **Path Parameters**: `section_id` (integer)
- **Request Body** (all fields optional):
  ```json
  {
    "name": "string|null",
    "code": "string|null",
    "class_id": 1|null,
    "description": "string|null"
  }
  ```
- **Response**: `200 OK` (section detail response)

### 6.5 Delete Section
- **Endpoint**: `DELETE /sections/{section_id}`
- **Auth Required**: Yes
- **Path Parameters**: `section_id` (integer)
- **Response**: `204 No Content`

---

## 7. Department APIs

### 7.1 List Departments
- **Endpoint**: `GET /departments/`
- **Auth Required**: Yes
- **Query Parameters**:
  - `page` (optional, default: 1, min: 1)
  - `page_size` (optional, default: 100, min: 1, max: 1000)
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "data": [
      {
        "id": 1,
        "name": "string",
        "description": "string|null",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "message": "Operation successful"
  }
  ```

### 7.2 Get Department
- **Endpoint**: `GET /departments/{department_id}`
- **Auth Required**: Yes
- **Path Parameters**: `department_id` (integer)
- **Response**: `200 OK` (department detail response)

### 7.3 Create Department
- **Endpoint**: `POST /departments/`
- **Auth Required**: Yes
- **Request Body**:
  ```json
  {
    "name": "string",
    "description": "string|null"
  }
  ```
- **Response**: `201 Created` (department detail response)

### 7.4 Update Department
- **Endpoint**: `PUT /departments/{department_id}`
- **Auth Required**: Yes
- **Path Parameters**: `department_id` (integer)
- **Request Body** (all fields optional):
  ```json
  {
    "name": "string|null",
    "description": "string|null"
  }
  ```
- **Response**: `200 OK` (department detail response)

### 7.5 Delete Department
- **Endpoint**: `DELETE /departments/{department_id}`
- **Auth Required**: Yes
- **Path Parameters**: `department_id` (integer)
- **Response**: `204 No Content`

---

## 8. Role APIs

### 8.1 List Roles
- **Endpoint**: `GET /roles/`
- **Auth Required**: Yes
- **Query Parameters**:
  - `page` (optional, default: 1, min: 1)
  - `page_size` (optional, default: 100, min: 1, max: 1000)
- **Response**: `200 OK`
  ```json
  {
    "success": true,
    "data": [
      {
        "id": 1,
        "name": "string",
        "description": "string|null",
        "created_at": "datetime",
        "updated_at": "datetime"
      }
    ],
    "message": "Operation successful"
  }
  ```

### 8.2 Get Role
- **Endpoint**: `GET /roles/{role_id}`
- **Auth Required**: Yes
- **Path Parameters**: `role_id` (integer)
- **Response**: `200 OK` (role detail response)

### 8.3 Create Role
- **Endpoint**: `POST /roles/`
- **Auth Required**: Yes
- **Request Body**:
  ```json
  {
    "name": "string",
    "description": "string|null"
  }
  ```
- **Response**: `201 Created` (role detail response)

### 8.4 Update Role
- **Endpoint**: `PUT /roles/{role_id}`
- **Auth Required**: Yes
- **Path Parameters**: `role_id` (integer)
- **Request Body** (all fields optional):
  ```json
  {
    "name": "string|null",
    "description": "string|null"
  }
  ```
- **Response**: `200 OK` (role detail response)

### 8.5 Delete Role
- **Endpoint**: `DELETE /roles/{role_id}`
- **Auth Required**: Yes
- **Path Parameters**: `role_id` (integer)
- **Response**: `204 No Content`

---

## API Summary

### Total Endpoints: 36

- **Authentication**: 5 endpoints (2 login endpoints, logout, refresh, verify)
- **Health**: 1 endpoint
- **Schools**: 4 endpoints (onboard, list, get, update)
- **Staff**: 5 endpoints
- **Classes**: 5 endpoints
- **Sections**: 5 endpoints
- **Departments**: 5 endpoints
- **Roles**: 5 endpoints

### Common Patterns

1. **Authentication**: All endpoints (except login) require Bearer token authentication
2. **Pagination**: List endpoints support pagination with `page` and `page_size` parameters
3. **CRUD Operations**: Standard Create, Read, Update, Delete operations
4. **Response Format**: Consistent response format with `success`, `data`, and `message` fields
5. **Timestamps**: All timestamps are in ISO 8601 datetime format

### Error Responses

- **422 Validation Error**: Invalid request data
- **403 Forbidden**: Insufficient permissions
- **401 Unauthorized**: Missing or invalid token
- **404 Not Found**: Resource not found

### Field Validation Rules

- **Email**: Must be valid email format
- **Password**: Minimum 6 characters (for staff creation)
- **String Lengths**: 
  - Names: 1-255 characters
  - Codes: 1-50 characters
  - Phone: Max 20 characters
  - Address: Max 500 characters
- **IDs**: Must be positive integers (> 0)
- **School Status**: Enum values: `active`, `inactive`, `suspended`, `trial`

### Notes for UI Integration

1. **Token Management**: Store JWT token securely and include in `Authorization` header for all authenticated requests
2. **Login Endpoints**: Use `/auth/login/internal` for admin/internal employees, `/auth/login/staff` for school staff
3. **Token Refresh**: Implement automatic token refresh using `/auth/refresh` before token expiration
4. **Error Handling**: Handle 401 errors by redirecting to login page
5. **Pagination**: Implement pagination controls for all list endpoints
6. **Form Validation**: Validate required fields and formats on client-side before API calls
7. **Loading States**: Show loading indicators during API calls
8. **Success/Error Messages**: Display appropriate messages based on API responses
9. **School Onboarding**: Use `/schools/onboard` endpoint to create new schools (internal employees only)

### Additional Resources

- **Swagger Documentation**: Available at `/docs/swagger.yaml` or `/docs/swagger.json`
- **Interactive API Docs**: Available at `/docs` endpoint when server is running

