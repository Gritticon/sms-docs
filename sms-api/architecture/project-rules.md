---
title: Project Development Rules
type: architecture
project: sms-api
last-updated: 2026-03-18
---

# Project Development Rules

This document outlines the mandatory development rules for the SMS API project.

## 1. No Raw SQL Queries

**Rule**: No SQL queries should be written directly in the codebase.

**Rationale**: Using raw SQL bypasses the benefits of SQLAlchemy ORM, including:
- Type safety
- SQL injection protection
- Database abstraction
- Query optimization
- Relationship management

**Implementation**:
- Use SQLAlchemy ORM methods exclusively
- Leverage `session.query()`, `session.add()`, `session.delete()`, etc.
- Use SQLAlchemy's query builder: `select()`, `update()`, `delete()`
- For complex queries, use SQLAlchemy expressions and joins

**Examples**:

❌ **BAD**:
```python
result = session.execute("SELECT * FROM customers WHERE id = %s", (customer_id,))
```

✅ **GOOD**:
```python
result = session.query(Customer).filter(Customer.id == customer_id).first()
```

## 2. Priority to Packages and ORM

**Rule**: Priority goes to packages. SQLAlchemy provides benefits, but writing raw queries doesn't make sense.

**Rationale**: Established packages provide:
- Battle-tested code
- Security features
- Performance optimizations
- Maintainability
- Community support

**Implementation**:
- Always prefer SQLAlchemy ORM over manual query construction
- Use SQLAlchemy relationships (`relationship()`) for foreign keys
- Leverage lazy loading, eager loading, and other ORM features
- Use SQLAlchemy's session management and transaction handling

**Examples**:

❌ **BAD**:
```python
# Manual join logic
customers = session.execute("""
    SELECT c.*, d.name as dept_name 
    FROM customers c 
    JOIN departments d ON c.dept_id = d.id
""")
```

✅ **GOOD**:
```python
# Using ORM relationships
customers = session.query(Customer).join(Department).all()
# Or using relationship
customer = session.query(Customer).first()
dept_name = customer.department.name  # Uses relationship
```

## 3. Remove Unused and Redundant Code

**Rule**: Actively identify and remove unused and redundant code before committing.

**Rationale**: Unused and redundant code:
- Increases maintenance burden
- Reduces code clarity
- Can cause confusion
- Increases technical debt
- Makes codebase harder to navigate
- Can lead to security issues (unused dependencies)

**Implementation**:
- **Remove unused code**:
  - Unused imports, functions, classes, variables, and parameters
  - Dead code paths and unreachable code
  - Commented-out code blocks (unless they serve a clear documentation purpose)
  - Unused files and scripts
- **Remove redundant code**:
  - Duplicate functionality across files
  - Files that duplicate what ORM/models already provide (e.g., SQL files for tables that SQLAlchemy auto-creates)
  - Redundant patterns or duplicate logic
  - Unnecessary abstraction layers
- Use linters and static analysis tools (pylint, flake8, mypy, vulture) to identify unused code
- Before committing, scan for:
  - Unused imports (verify all `import` and `from` statements are used)
  - Unused functions/methods that are never called
  - Unused classes that are never instantiated
  - Unused variables and parameters
  - Redundant files (SQL scripts that duplicate ORM functionality, duplicate utility scripts)
- If code is temporarily unused but may be needed later, add a `# TODO: Remove if not used by [date]` comment

**Examples**:

❌ **BAD**:
```python
from typing import List, Dict, Tuple  # Tuple is unused
import json  # json is unused

def unused_function():
    pass

# Old implementation - commented out
# def old_way():
#     pass
```

❌ **BAD** - Redundant SQL file:
```sql
-- create_table.sql (redundant if SQLAlchemy auto-creates tables)
CREATE TABLE IF NOT EXISTS customers (...);
```

✅ **GOOD**:
```python
from typing import List, Dict

# Only keep imports that are actually used
```

✅ **GOOD** - Remove redundant SQL file:
- SQLAlchemy model already exists in `app/models/customer.py`
- Table is auto-created via `Base.metadata.create_all()`
- SQL file is redundant and should be removed

## 4. One Model/Schema Per File

**Rule**: No two models or schemas should be written on a single page.

**Rationale**: Separation improves:
- Code organization
- Maintainability
- Readability
- Reduces merge conflicts
- Makes navigation easier

**Implementation**:
- Each model class in `app/models/` must be in its own file
- Each schema class in `app/schemas/` must be in its own file
- File naming: `model_name.py` for models, `schema_name.py` for schemas
- Follow existing pattern: `customer.py`, `staff.py`, `department.py`, etc.

**Examples**:

❌ **BAD** (`app/models/customer.py`):
```python
class Customer(Base):
    # ...

class CustomerAddress(Base):
    # ...
```

✅ **GOOD**:
- `app/models/customer.py` - contains only `Customer` model
- `app/models/customer_address.py` - contains only `CustomerAddress` model

## 5. Safe Coding Practices

**Rule**: Always follow safe written practices like try-catch, rollback on failure.

**Rationale**: Database operations can fail due to:
- Network issues
- Constraint violations
- Data validation errors
- Concurrent access conflicts

**Implementation**:
- Always wrap database operations in try-except blocks
- Implement transaction rollback on exceptions
- Use context managers for session management when possible
- Provide meaningful error messages
- Log errors appropriately

**Examples**:

❌ **BAD**:
```python
def create_customer(data):
    customer = Customer(**data)
    session.add(customer)
    session.commit()  # No error handling
```

✅ **GOOD**:
```python
def create_customer(data):
    try:
        customer = Customer(**data)
        session.add(customer)
        session.commit()
        return customer
    except Exception as e:
        session.rollback()
        logger.error(f"Failed to create customer: {str(e)}")
        raise
    finally:
        session.close()
```

**Better** (using context manager):
```python
def create_customer(data):
    try:
        with session.begin():
            customer = Customer(**data)
            session.add(customer)
            return customer
    except Exception as e:
        logger.error(f"Failed to create customer: {str(e)}")
        raise
```

## 6. Verify Application After Changes

**Rule**: After every change in files, verify the application runs without errors.

**Rationale**: 
- Catches errors early before they propagate
- Ensures code changes don't break existing functionality
- Prevents committing broken code
- Saves time by fixing issues immediately

**Implementation**:
- **MANDATORY**: After every file change, verify the application:
  - Starts without errors
  - Has no import errors
  - Has no syntax errors
  - Has no runtime errors
  - Passes linter checks
- Check for errors before proceeding with more changes
- Fix any errors immediately
- Use the following methods to verify:
  - Run the application: `uvicorn main:app --reload`
  - Check syntax: `python -m py_compile app/main.py`
  - Run linters: `pylint app/` or `flake8 app/`
  - Check imports are valid

**Examples**:

❌ **BAD**:
- Making multiple changes without testing
- Committing code that hasn't been verified
- Ignoring import or syntax errors

✅ **GOOD**:
```bash
# After making changes, verify app runs:
uvicorn main:app --reload

# Or check for errors:
python -m py_compile app/main.py
python -c "import app.main"
```

## 7. Check and Fix Script Errors

**Rule**: **MANDATORY** - All responses must check scripts in the `scripts/` directory for errors and fix them.

**Rationale**: 
- Scripts are critical utilities used for setup, maintenance, and operations
- Broken scripts can prevent deployment, setup, or maintenance tasks
- Script errors can cause production issues
- Ensures all scripts follow project rules (no raw SQL, proper error handling, etc.)

**Implementation**:
- **MANDATORY**: After any code changes, check all scripts in `scripts/` directory:
  - Verify scripts compile without syntax errors
  - Check for rule violations (raw SQL, unused imports, etc.)
  - Ensure proper error handling (try-except blocks, rollback)
  - Verify scripts can be imported/executed without errors
  - Fix any errors found immediately
- Use the following methods to verify scripts:
  - Check syntax: `python -m py_compile scripts/*.py`
  - Test imports: `python -c "import scripts.create_internal_employee"`
  - Run linters on scripts: `pylint scripts/` or `flake8 scripts/`
  - Verify scripts follow all project rules (no raw SQL, proper error handling, etc.)
- Scripts must follow the same rules as application code:
  - No raw SQL queries (use SQLAlchemy ORM)
  - Proper error handling with try-except and rollback
  - No unused imports or code
  - Safe coding practices

**Examples**:

❌ **BAD**:
```python
# Script with raw SQL
def create_user():
    db.execute("INSERT INTO users VALUES (...)")
```

❌ **BAD**:
```python
# Script without error handling
def create_user():
    user = User(...)
    db.add(user)
    db.commit()  # No try-except, no rollback
```

✅ **GOOD**:
```python
# Script following all rules
def create_user():
    db = SessionLocal()
    try:
        user = User(...)
        db.add(user)
        db.commit()
        return user
    except Exception as e:
        db.rollback()
        print(f"Error: {e}")
        raise
    finally:
        db.close()
```

**Verification Checklist**:
- ✅ All scripts compile without syntax errors
- ✅ No raw SQL queries in scripts
- ✅ All database operations have try-except with rollback
- ✅ No unused imports or code in scripts
- ✅ Scripts can be imported/executed without errors
- ✅ Scripts follow all project rules

## 8. Code Review Checklist

Before committing code, verify:
- ✅ No unused imports, functions, classes, or variables
- ✅ No redundant files or duplicate functionality
- ✅ No commented-out code blocks (unless clearly marked as documentation)
- ✅ No SQL files that duplicate ORM table creation
- ✅ No duplicate implementations of the same functionality
- ✅ All code paths are reachable and used
- ✅ All imported modules/packages are actually used
- ✅ Application runs without errors after changes
- ✅ All scripts in `scripts/` directory are checked and error-free
- ✅ Scripts follow all project rules (no raw SQL, proper error handling, etc.)

## Enforcement

These rules should be:
- Reviewed during code reviews using the checklist above
- Checked by linters and static analysis tools (pylint, flake8, mypy, vulture)
- Documented in pull request templates
- Enforced by CI/CD pipelines where possible
- Regularly audit the codebase for redundant and unused code

## Related Documentation

- [Development Guide](./development.md)
- [Deployment Guide](./deployment.md)

