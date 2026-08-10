# Implement `POST /api/v1/auth/register`

You are working inside the existing **Manage-R backend** codebase.

## Objective

Implement the first real authentication feature:

```http
POST /api/v1/auth/register
```

Follow the **existing project architecture and conventions**. Do not redesign the foundation or introduce a new architecture.

Before changing anything, inspect:

- `src/common/**`
- `src/config/**`
- `src/database/**`
- `src/modules/auth/**`
- `src/modules/users/**`
- `src/modules/roles/**`
- Prisma schema
- Existing DTOs, repositories, services, guards, decorators, exceptions, interceptors, constants
- Existing response/error format
- Existing validation and logging conventions
- `manage-r-docs/guides/backend/write-business-code.md` if available

Also inspect:

```text
/Users/tasinsc/Learning/manage-r/manage-r-documents/work-progress.txt
/Users/tasinsc/Learning/manage-r/manage-r-documents/guides/backend/write-features.txt
```

Preserve the existing coding style.

---

## Functional requirements

Implement registration with this flow:

```text
HTTP Request
    ↓
Controller
    ↓
DTO validation
    ↓
Auth Service
    ↓
Check existing user
    ↓
Validate/default user role
    ↓
Hash password
    ↓
Create user
    ↓
Create associated profile if required by the existing schema/design
    ↓
Return sanitized response
```

### Request

Use the project's existing DTO/validation conventions.

The registration request should contain the minimum fields required by the existing database design, such as:

```json
{
  "username": "tasin",
  "email": "tasin@example.com",
  "phoneNumber": "017XXXXXXXX",
  "password": "StrongPasswordHere"
}
```

Do not expose or accept privileged fields such as:

```text
userId
role
passwordHash
createdAt
updatedAt
```

The role must **never be supplied by the client**.

Use the normal/default user role from the existing role system.

---

## Validation

Implement appropriate validation using the project's existing `class-validator` setup.

At minimum:

- username required
- email required and valid
- password required
- strong-password validation using the project's existing validator
- phone number validation if the existing DTO/database rules require it

Do not duplicate validation logic unnecessarily.

---

## Duplicate handling

Before creating the user, check whether the relevant unique fields already exist.

At minimum consider:

```text
username
email
phone_number
```

Use the database's unique constraints as the final protection against duplicates.

Return the project's standard conflict/business exception instead of leaking raw Prisma/database errors.

Handle Prisma unique-constraint errors properly.

---

## Password security

Never store the plain-text password.

Use the project's existing hashing utility if one already exists.

If the current implementation uses bcrypt:

```text
plain password
      ↓
bcrypt hash
      ↓
password_hash
```

Never return:

```text
password
passwordHash
```

in the API response.

---

## Repository

Follow the existing repository abstraction.

The repository should be responsible for persistence/database access.

Do not put Prisma queries directly inside the controller.

Keep responsibilities separated:

```text
Controller → HTTP concerns
Service    → business logic
Repository → database operations
DTO        → input validation
Mapper     → response transformation, if used
```

Reuse existing repository interfaces/base abstractions where appropriate.

---

## Response

Follow the existing standard API response structure.

Return the newly created user's safe information only.

Example shape:

```json
{
  "success": true,
  "data": {
    "userId": 1,
    "username": "tasin",
    "email": "tasin@example.com"
  }
}
```

Do not expose:

- password
- password hash
- internal security information
- unnecessary database fields

Do not automatically log sensitive registration data.

---

## Authentication behavior

Registration itself does not need to return access/refresh tokens unless the existing authentication design explicitly requires it.

Keep registration and login responsibilities separate.

The next feature will be:

```text
POST /api/v1/auth/login
```

---

## Database considerations

Use the existing Prisma schema and existing database.

Do not modify the database schema unless the implementation genuinely requires it.

If a schema problem is discovered:

1. Stop.
2. Explain the problem.
3. Make the smallest safe change.
4. Do not silently redesign the schema.

---

## Error handling

Use the project's existing exception hierarchy and global exception filter.

Handle at least:

```text
400 → invalid input
409 → duplicate username/email/phone
500 → unexpected database/application error
```

Do not return raw Prisma errors to clients.

---

## Testing

After implementation:

1. Run TypeScript/build checks.
2. Run ESLint.
3. Run existing tests.
4. Add focused tests for registration if the project already has a testing convention.

At minimum verify:

```text
✓ valid registration succeeds
✓ invalid email is rejected
✓ weak password is rejected
✓ duplicate email is rejected
✓ duplicate username is rejected
✓ password is hashed
✓ password/hash is not returned
✓ role cannot be injected by the client
✓ database errors are converted to appropriate application errors
```

Do not weaken ESLint rules merely to make the implementation pass.

---

# Documentation requirements

After the implementation is complete, update:

```text
/Users/tasinsc/Learning/manage-r/manage-r-documents/work-progress.txt
```

Mark:

```text
POST /api/v1/auth/register
```

as:

```text
✅ DONE
```

Preserve the existing formatting of the file.

Do not rewrite unrelated progress entries.

---

## Write the implementation flow

Also update:

```text
/Users/tasinsc/Learning/manage-r/manage-r-documents/guides/backend/write-features.txt
```

Add a **short, beginner-friendly explanation** of how this feature works.

Keep it concise. This is a personal learning/reference document, not a formal textbook.

Explain the flow:

```text
Request
  ↓
DTO
  ↓
Controller
  ↓
Service
  ↓
Repository
  ↓
Prisma
  ↓
PostgreSQL
  ↓
Response
```

Also briefly explain:

### 1. DTO

Show the essential DTO snippet.

### 2. Controller

Show the essential controller snippet.

### 3. Service

Show the important business-logic snippet:

```text
check duplicate
→ hash password
→ create user
→ sanitize result
```

### 4. Repository

Show the essential database-operation snippet.

### 5. Database

Briefly show how the repository reaches the Prisma model/PostgreSQL.

### 6. Request/response

Show one small example:

```http
POST /api/v1/auth/register
```

with a small JSON request and response.

### 7. Security

Briefly explain:

```text
Why password is hashed
Why role is not accepted from the client
Why passwordHash is never returned
Why duplicate checks exist
```

Use **only essential code snippets**.

Do not dump complete source files into the guide.

The explanation should be understandable to someone who has basic programming knowledge but is rusty with NestJS.

---

# Important constraints

- Do NOT rewrite the existing architecture.
- Do NOT introduce unnecessary packages.
- Do NOT create duplicate utilities.
- Do NOT bypass repositories.
- Do NOT put business logic in controllers.
- Do NOT expose Prisma entities directly as API responses.
- Do NOT accept `role` from registration input.
- Do NOT store plain passwords.
- Do NOT expose password hashes.
- Do NOT modify unrelated modules.
- Do NOT refactor the entire codebase.
- Follow existing project conventions over personal preferences.

At the end, report:

```text
Implementation:
- files created/modified
- endpoint implemented
- tests/checks executed
- documentation updated

Result:
POST /api/v1/auth/register → DONE
```
