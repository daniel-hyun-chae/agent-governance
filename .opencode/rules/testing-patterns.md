# Testing Patterns

Architecture-level checklists for common categories. Apply these when implementing or reviewing tests in the relevant area. These patterns complement, not replace, the spec-level behaviors.

## API Resolvers

- Auth happy-path (valid token, correct role) returns expected data.
- Missing / invalid / expired token returns 401 or appropriate error.
- Requesting a resource that does not exist returns a not-found error.
- Invalid input (wrong types, missing required fields) returns a validation error.
- Valid token but wrong role returns a forbidden error.

## Repository / Data-Access Functions

- Happy-path returns the expected row(s).
- Querying a non-existent ID returns null or empty, not an exception.
- Violating a unique / foreign-key constraint returns a typed error.
- Filters, sort, and pagination return correct subsets.
- Queries behave identically against the real database (integration) and in-memory fixtures (unit), or differences are documented.

## UI Components -- Interactive Behavior

- Component renders in its default state without errors.
- User interaction (click, type, select) triggers the expected state change or callback.
- Loading state is shown while async data is in flight.
- Error state is shown when the data fetch fails.
- Keyboard navigation (Tab, Enter, Escape) works for all interactive elements.
- Accessibility: no violations reported by axe or equivalent automated checker.

## Auth Guards and Middleware

- Valid token passes through and attaches the expected context.
- Expired token is rejected with a clear error.
- Missing token is rejected (not silently ignored).
- Malformed token is rejected (not parsed as valid).
- Insufficient role is rejected even with a valid token.

## Data Migrations and Seed Scripts

- Running on a fresh database produces the expected schema and seed data.
- Running incrementally (on an already-migrated database) is idempotent.
- Rollback (if supported) restores the previous state without data loss.
- Backward-compatible: the previous application version can still read data after migration.

## When to Apply

These checklists are not rigid requirements for every test file. Use judgment: if the feature under test involves an API resolver, walk through the API Resolver checklist. The goal is a consistent safety net, not bureaucratic coverage.

Add product-specific testing patterns in `project/.opencode-overrides/rules/` as needed.
