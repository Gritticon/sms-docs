---
title: Troubleshooting 401 Authentication Errors
type: setup
project: sms-api
last-updated: 2026-03-18
---

# Troubleshooting 401 Authentication Errors

## Issue: "Token is required" Error

If you're getting a 401 error saying "Token is required", follow these steps:

### Step 1: Verify Token is Being Sent

1. **Use the test endpoint** to see what headers are being received:
   - Go to `/docs`
   - Call `POST /auth/test-token`
   - **Before calling**, click "Authorize" and enter your token
   - Check the response to see if the Authorization header is present

### Step 2: Check How You're Entering the Token in Swagger

**Correct way:**
1. Click the **"Authorize"** button (lock icon, top right)
2. In the "Value" field, enter **ONLY the token** (without "Bearer ")
3. Example: If your token is `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`, enter just that
4. Swagger automatically adds "Bearer " prefix

**Wrong ways:**
- ❌ Entering `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` (double Bearer)
- ❌ Entering token with extra spaces or newlines
- ❌ Not clicking "Authorize" at all

### Step 3: Verify Token is Valid

1. **Login again** to get a fresh token:
   ```
   POST /auth/login
   {
     "school_id": 0,
     "email": "admin@example.com",
     "password": "admin123"
   }
   ```

2. **Copy the entire token** from the response (they're long!)

3. **Use it immediately** - tokens can expire

### Step 4: Check the Error Message

The improved error message will now show:
- What headers are available
- Whether Authorization header is present
- More specific error details

Look for messages like:
- "Token has expired" → Login again
- "Invalid token signature" → Token corrupted, login again
- "Token missing user_id" → Token format issue, login again

### Step 5: Manual Header Test

If Swagger isn't working, test with curl:

```bash
# First, login to get token
curl -X POST "http://localhost:8000/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "school_id": 0,
    "email": "admin@example.com",
    "password": "admin123"
  }'

# Copy the token from response, then test:
curl -X GET "http://localhost:8000/auth/verify" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

### Common Issues

1. **Token not copied completely**
   - Tokens are very long (200+ characters)
   - Make sure you copy the entire token

2. **Token expired**
   - Default expiry is 30 days
   - Solution: Login again

3. **Swagger UI cache**
   - Try refreshing the page
   - Clear browser cache
   - Try in incognito/private mode

4. **Header not being sent**
   - Check browser developer tools → Network tab
   - Look at the request headers
   - Verify "Authorization: Bearer ..." is present

### Debug Endpoint

Use `/auth/test-token` to see:
- What headers are being received
- Whether Authorization header exists
- The actual header values (truncated for security)

This helps diagnose if the issue is:
- Token not being sent
- Header name mismatch
- Token format issue

