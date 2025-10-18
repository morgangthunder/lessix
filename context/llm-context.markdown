# AI Lessons App MVP Context

## Overview
Build an AI lessons app (MVP) with an Ionic/Angular frontend, NestJS backend, PostgreSQL (AWS RDS with Row-Level Security for multi-tenancy via `tenant_id`), Amazon S3 for asset storage, xAI Grok API for AI-driven chat, content processing, and coding assistance, and n8n for workflow automation. Use an Nx monorepo, Docker/Coolify for local development, and AWS for production. Prioritize simplicity for a non-experienced coder, ensuring robust testing and validation.

## Key Requirements
- **Frontend**: Ionic/Angular app with a Netflix-style Lesson Browser (card-based grid), Lesson-Builder (form with JSON editor, submit-for-approval button, status display: `pending/approved/rejected`), and Lesson View (Pixi.js for interactions, Grok API chat). Support Angular Elements to package the app as a web component (JavaScript blob) for embedding in external websites, configurable via environment variables (e.g., `ACCOUNT_ID` to filter lessons by account, `TENANT_ID` for tenant-specific data, `PUBLIC_MODE=true` to show all approved lessons from all creators).
- **Backend**: NestJS with REST APIs (e.g., `POST /lessons/submit` for lesson submission with `tenant_id`, `account_id`, and `status=pending`). Support querying all approved lessons across accounts/tenants for a public instance when `PUBLIC_MODE=true`. Use PostgreSQL with RLS for tenant isolation, and Socket.io for real-time notifications (e.g., approval status updates).
- **Authentication**: Default to AWS Cognito for user authentication (JWT-based). Support external authentication (e.g., company SSO via OAuth2, SAML, or custom JWTs) by accepting external tokens, validating them, and extracting `tenant_id` and `account_id` for access control. Allow toggling via environment variable (e.g., `AUTH_PROVIDER=cognito|external|none`). For public instances (`PUBLIC_MODE=true`), support unauthenticated/guest access to approved lessons.
- **Approval Process**: All lessons, interaction types, and n8n workflows require approval (`pending/approved/rejected`). Only approved content is accessible, enforced via backend APIs and RLS, including in public mode.
- **Multi-Tenancy**: Isolate data by `tenant_id` using PostgreSQL RLS. Environment variables (e.g., `TENANT_ID`, `ACCOUNT_ID`, `PUBLIC_MODE`) configure app instances for specific tenants, accounts, or public access.
- **Storage**: Use Amazon S3 (mocked with MinIO locally) for lesson assets (e.g., images, JSON).
- **AI Integration**: Use xAI Grok API for AI chat, content generation, and coding assistance. Track token usage for cost monitoring.
- **Workflows**: Use n8n for approval workflows, triggered by lesson submissions and status changes.
- **Payments**: Implement Stripe Connect for creator commissions, tied to approved lessons.
- **Testing**: Use Jest for frontend unit tests, Mocha/Chai for backend unit tests, and Cypress for E2E browser tests. Validate JSON schemas for lesson data and API payloads.
- **Development**: Use Nx monorepo with esbuild, Sass, no SSR/SSG, Jest, Cursor/GitHub Copilot AI agents, and GitHub Actions for CI. Use Docker/Coolify for local dev and AWS for production.
- **Simplicity**: Prioritize clear code, minimal setup complexity, and Cursor-generated code/tests for a non-experienced coder.

## Critical Analysis
- Critically analyze all requests to ensure alignment with the stack and requirements.
- Flag issues (e.g., skipping approval logic, missing tenant isolation) and suggest alternatives.
- Ensure environment variables (e.g., `ACCOUNT_ID`, `TENANT_ID`, `PUBLIC_MODE`, `AUTH_PROVIDER`) are used to configure Angular Elements and authentication for enterprise use cases (e.g., embedding in external sites, using company SSO) and public instances (showing all approved lessons).

## Notes
- Full context is in `/context/llm-context.md`.
- Use the React demo repo (https://github.com/morgangthunder/BloomixDemo, e.g., "card-based lesson grid, JSON editor form") as a UI reference, adapted to Ionic/Angular.
- For Angular Elements, ensure the app can run as a standalone web component with minimal dependencies, configurable via environment variables.
- For public instances, use `PUBLIC_MODE=true` to show all approved lessons, with `AUTH_PROVIDER=none` for guest access or custom auth.
- For external authentication, design APIs to accept and validate external tokens, mapping to `tenant_id` and `account_id`.