# AGENTS.md — build-with-paragon Skill

## Repository Purpose

This is a **documentation-only skill repository** for agentic coding IDEs (Cursor, Claude Code, Opencode).
It contains no source code, build tooling, tests, or CI/CD. Its purpose is to provide reference
material so AI coding agents can help developers integrate with the Paragon platform.

## Build / Lint / Test Commands

There are no build, lint, or test commands in this repository. All content is Markdown.
If you are editing reference docs, validate links manually and ensure code examples are syntactically correct.

## Repository Structure

```
.
├── SKILL.md                              # Skill manifest (name, description, table of contents)
├── README.md                             # Installation instructions for IDE skill directories
├── AGENTS.md                             # This file
└── references/
    ├── paragon-sdk-setup.md              # SDK install, JWT signing, paragon.authenticate()
    ├── connect-integrations.md           # Integration metadata, Connect Portal rendering
    ├── actionkit.md                      # LIST ACTIONS / RUN ACTION endpoints, AI agent tools example
    ├── managed-sync.md                   # Enable Sync, Pull Records, Download Files, Permissions API
    └── sync-management.md               # List / Disable / Re-enable / Delete sync pipelines
```

## How This Skill Is Used

When a user asks about building integrations with Paragon, the agent should:

1. Load the skill via the `build-with-paragon` skill loader.
2. Consult `SKILL.md` for the table of contents to find the right reference.
3. Read the relevant `references/*.md` file(s) for API details, code examples, and checklists.

## Paragon API & SDK Quick Reference

### Key Packages

| Package | Purpose |
|---|---|
| `@useparagon/connect` | Frontend SDK — authenticate users, render Connect Portal |
| `jsonwebtoken` | Server-side JWT signing for Paragon user tokens |

### API Base URLs

| Service | Base URL |
|---|---|
| ActionKit | `https://actionkit.useparagon.com/projects/{project_id}/actions` |
| Managed Sync | `https://sync.useparagon.com/api/syncs` |
| Permissions | `https://sync.useparagon.com/api/permissions/{syncId}/...` |

### Authentication

All API requests require a `Authorization: Bearer {PARAGON_JWT}` header.
The JWT must be signed server-side with RS256 using the project's Signing Key.

**JWT payload structure:**
```json
{
  "sub": "<user-or-company-id>",
  "aud": "useparagon.com/<paragon-project-id>",
  "iat": <unix-timestamp>,
  "exp": <unix-timestamp>
}
```

**Critical:** Never expose the Signing Key on the client side.

## Code Style Guidelines for Paragon Integrations

When generating code that uses Paragon APIs/SDK, follow these conventions:

### Imports

- Use ES module imports: `import { paragon } from '@useparagon/connect'`
- Import specific types when using TypeScript: `import { paragon, IntegrationMetadata } from '@useparagon/connect'`
- Group imports: third-party packages first, then local modules

### TypeScript & Types

- Prefer TypeScript over plain JavaScript for all new code
- Use explicit types for Paragon SDK return values (e.g., `IntegrationMetadata[]`)
- Avoid `any` where possible; use it only as a last resort in dynamic tool parameter contexts
- Use `interface` for object shapes, `type` for unions and intersections

### Async / Await

- Always use `async/await` over raw Promises for readability
- All Paragon SDK calls (`paragon.authenticate`, `paragon.getIntegrationMetadata`) are async

### Error Handling

- Wrap all API calls (fetch, SDK methods) in `try/catch` blocks
- Use `instanceof Error` checks before accessing `.message`:
  ```typescript
  try {
    const res = await fetch(url, options);
    const data = await res.json();
    if (!res.ok) {
      throw new Error(JSON.stringify(data, null, 2));
    }
    return data;
  } catch (err) {
    if (err instanceof Error) {
      return { error: { message: err.message } };
    }
    return err;
  }
  ```
- Check `response.ok` after every `fetch` call; throw on non-2xx responses
- For ActionKit LIST ACTIONS, check `errors.length === 0` before proceeding

### Naming Conventions

- **Variables/functions:** camelCase (`paragonUserToken`, `getIntegrationMetadata`)
- **Constants/env vars:** UPPER_SNAKE_CASE (`PARAGON_PROJECT_ID`, `PARAGON_SIGNING_KEY`)
- **ActionKit action names:** UPPER_SNAKE_CASE with integration prefix (`SLACK_SEND_DIRECT_MESSAGE`, `SALESFORCE_UPDATE_RECORD`)
- **Integration type identifiers:** lowercase, no separators (`salesforce`, `hubspot`, `googledrive`)

### Environment Variables

Always store secrets as environment variables, never hardcode them:
- `PARAGON_PROJECT_ID` — the project identifier from the Paragon dashboard
- `PARAGON_SIGNING_KEY` — the RS256 private key for JWT signing

### API Request Patterns

- Always set `Content-Type: application/json` on POST requests
- Use template literals for URL construction with project IDs
- Pass the Paragon JWT via the `Authorization: Bearer` header

### Frontend / React Patterns

- Use the Paragon SDK methods (`paragon.connect(type)`) for OAuth flows, not raw API calls
- Render integration catalogs using `paragon.getIntegrationMetadata()` results
- For on-prem deployments, call `paragon.configureGlobal({ host })` before `authenticate`

## Common Pitfalls

1. **Signing JWT client-side** — The Signing Key must only be used in server-side code.
2. **Missing authentication** — `paragon.authenticate()` must be called before any other SDK method.
3. **Forgetting `format` param** — For AI agent tools, use `format=json_schema` on LIST ACTIONS.
4. **Wrong action parameters** — Always fetch available actions first via LIST ACTIONS; use the returned parameter schema to build RUN ACTION requests.
5. **Not checking response status** — Always verify `response.ok` and handle errors from both ActionKit and Managed Sync APIs.

## Reference Navigation Guide

| User Need | Start Here |
|---|---|
| New to Paragon, no SDK setup yet | `references/paragon-sdk-setup.md` |
| Need to connect user integrations (OAuth) | `references/connect-integrations.md` |
| CRUD actions on 3rd-party data | `references/actionkit.md` |
| Bulk data ingestion / ETL pipelines | `references/managed-sync.md` |
| Managing sync lifecycle (pause/resume/delete) | `references/sync-management.md` |
| Building AI agent tools with integrations | `references/actionkit.md` (Example Implementation section) |
