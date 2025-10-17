# Context for AI Lessons App Development

You are assisting in building an AI lessons app with a focus on a minimal viable product (MVP) skeleton that proves end-to-end functionality. The app is a cross-platform, educational platform with a Netflix-style interface for students to take lessons, incorporating Pixi.js interactions and AI chat powered by xAI’s Grok API, and interfaces for lesson-builders and interaction-builders to create content. The goal is to create a modular, JSON-centric system that tracks Grok token usage, supports commissions for creators, processes content via n8n workflows, and supports enterprise clients with secure, private instances using multi-tenancy and Angular Elements-based web components. All lessons, interaction types, and n8n workflows require an approval process with submission statuses (e.g., pending, approved, rejected). The app must work locally using Docker Desktop and Coolify, deploying from GitHub, with a focus on separate component development and cross-platform deployment (browser, iOS via TestFlight, Android) using Ionic with Angular. The cloud provider is AWS, with Amazon Cognito for authentication and Amazon S3 for file storage. Below is the detailed context for all development tasks.

## App Features
1. **User Profiles and Authentication**:
   - Three account types: students (take lessons), lesson-builders (create lessons), interaction-builders (create Pixi.js interaction types and n8n workflows). Additional admin role per tenant for approving content.
   - Secure login with role-based access control (RBAC) via Amazon Cognito to enforce permissions (e.g., lesson-builders can’t access others’ earnings, admins approve content).
   - Profile pages showing usage metrics, commission earnings, and submission statuses (e.g., pending lessons/workflows) for builders.
   - Token tracking for Grok API usage (e.g., AI teacher chat, content processing, AI coding assistance) with warnings for approaching limits.

2. **Lesson Experience**:
   - Netflix-style interface for students to browse and take approved lessons only.
   - Lessons are JSON files defining stages, substages, source data links, processed data, Grok prompts for AI teachers, and interaction type configurations.
   - Lessons include Pixi.js interactions (e.g., drag-and-drop games) and real-time AI chat via WebSockets, powered by Grok API.
   - Students can interact with lessons (e.g., raise hand, pause) with real-time updates.

3. **Lesson-Builder Interface**:
   - Interface to structure lessons as JSON, including stages, substages, data links, Grok prompts for AI teachers, and interaction type selections.
   - Supports selecting approved interaction types and n8n workflows to process source content.
   - Includes a “Submit for Approval” button to send lessons for admin review, with status display (e.g., pending, approved, rejected).
   - Possible payment for premium interaction types.

4. **Interaction-Builder Interface**:
   - Interface to create Pixi.js interaction types, saved as JSON (defining inputs, events, and rendering configs).
   - Includes a code/preview split-view with Grok API assistance for coding Pixi.js interactions.
   - For MVP, the app owner creates initial n8n workflows offline using n8n’s native UI (e.g., localhost:5678) and uploads JSON via a simple file input for approval.
   - Interactions and workflows require admin approval before use, with status display.
   - Interactions run in a sandboxed Pixi.js environment, with defined input data structures and event triggers for lesson-builders to use (e.g., trigger AI teacher responses).

5. **Content Processing**:
   - Interface for uploading source content (e.g., text, PDFs, images) to S3, submitted for approval.
   - n8n-driven workflows (approved only) process content (e.g., extract text, summarize with Grok API, output JSON) and store results in S3 or Pinecone for semantic search.
   - Initial n8n workflows are created by the app owner, submitted for approval, and stored as JSON.

6. **Monetization**:
   - Commission system: Lesson-builders and interaction-builders earn based on usage of approved lessons/interactions/workflows only.
   - Usage metrics tracked in the database, aggregated for payouts via Stripe Connect.
   - Profile pages display earnings dashboards.

7. **Token Tracking**:
   - Tracks Grok API usage (e.g., AI chat, content processing, coding assistance) per user.
   - Backend middleware logs token usage, stores in DB with tenant isolation, and pushes warnings via WebSockets or in-app notifications when nearing limits.

8. **Enterprise Features**:
   - Support secure, private instances for enterprise clients using multi-tenancy to isolate data (e.g., lessons, processed content, usage metrics) per client.
   - Package the frontend as Angular Elements-based web components, bundled into a JavaScript blob that clients can embed on internal websites (e.g., `<lesson-viewer tenant="client-abc">`), showing only approved content.
   - Web components pull only tenant-specific, approved lessons and data, using segregated S3 buckets/prefixes and database rows (via tenant ID).
   - Ensure compatibility with enterprise requirements like GDPR/CCPA through data isolation and secure APIs.

9. **Approval Process**:
   - All lessons, interaction types, and n8n workflows must be submitted for admin approval before use.
   - Submission statuses (pending, approved, rejected) are tracked in the database and displayed in builder interfaces.
   - Tenant-specific admins (via Cognito groups) review and approve content, ensuring only approved content is accessible to students or enterprise users.
   - For MVP, the app owner manually approves content; post-MVP, add admin UI for tenant-specific approvals.

## App Component Structure
- **Frontend**: Ionic with Angular single-page app for dynamic UI rendering across browser, iOS (via TestFlight), and Android (via Capacitor). Convertible to Angular Elements for enterprise widget deployment.
  - Pages: Login, Profile (earnings, usage, submission statuses), Lesson Browser (Netflix-style, approved lessons only), Lesson View (Pixi.js + Grok-powered AI chat), Lesson-Builder (JSON editor with prompt management, interaction selection, submit button), Interaction-Builder (code/preview, JSON file upload for n8n workflows, submit button), Admin (post-MVP, for approving tenant-specific content).
  - Integrates Pixi.js for interactions and Socket.io for real-time AI chat and approval notifications.
- **Backend**: NestJS API server.
  - Endpoints: User management, lesson CRUD (with submit/approve), interaction type CRUD (with submit/approve), workflow CRUD (with submit/approve), content processing triggers, Grok token tracking, commission calculations, tenant-specific data access.
  - Webhook endpoints for n8n integration (e.g., trigger approved workflows, receive processed data).
- **Database**: PostgreSQL with JSONB for lessons, interaction types, n8n workflows, and Grok prompts. Supports multi-tenancy via Row-Level Security (RLS) with tenant IDs. Hosted on AWS RDS.
  - Tables: `users` (with `tenant_id`, role), `lessons` (JSONB for lesson data including prompts, `status` ENUM: pending/approved/rejected), `interaction_types` (JSONB for configs, `status`), `interaction_workflows` (JSONB for n8n workflows, `status`), `usages` (for metrics/commissions).
- **Vector Database**: Pinecone for indexing processed content and semantic search (e.g., lesson content, workflow outputs). Tenant-isolated data.
- **File Storage**: Amazon S3 with tenant-specific buckets/prefixes for source files and processed outputs.
- **Workflow Automation**: n8n for content processing and custom workflows by the app owner (for MVP).
  - Workflows stored as JSON, submitted for approval, triggered via API/webhooks, validated for security (e.g., allowed nodes only).
- **Real-time Features**: Socket.io for Grok-powered AI chat, lesson interactions, token warnings, and approval status updates, with tenant-namespaced rooms for enterprise isolation. Hosted via AWS API Gateway WebSockets.
- **Authentication**: Amazon Cognito with JWT-based auth, RBAC (e.g., tenant-admin group for approvals), and tenant claims. Supports enterprise OAuth federation for private instances.
- **Payment Integration**: Stripe Connect for commission payouts and potential interaction type payments.
- **Caching/Queuing**: Redis for caching lesson JSONs and queuing tasks (e.g., commission calculations, approval workflows). Tenant-aware caching. Hosted on AWS ElastiCache.
- **Monitoring/Logging**: Sentry for errors, Prometheus/Grafana for metrics, with tenant-specific logs. Integrated with AWS CloudWatch.
- **Testing**: Jest for frontend, Mocha/Chai for backend, Cypress for E2E tests, ensuring tenant isolation and approval flows.

## Technical Setup
- **Development Environment**: Local development with Docker Desktop and Coolify for deployment from GitHub. Production deployment on AWS (e.g., ECS for NestJS, RDS for PostgreSQL).
- **Monorepo**: Use Nx with folders: `/frontend` (Ionic/Angular, Angular Elements), `/backend` (NestJS), `/n8n` (workflows), `/shared` (types, e.g., lesson/interaction/workflow schemas).
- **Docker Compose**: Services for frontend (port 8100), backend (port 3000), PostgreSQL, n8n (port 5678, API/webhooks only), Pinecone (if local), Redis. Use MinIO for local S3 simulation.
- **Cross-Platform**: Ionic with Capacitor for browser, iOS (Xcode, TestFlight), Android (Android Studio). Angular Elements for enterprise web embedding.
- **AI Assistance**: Use Grok API (via https://x.ai/api) for AI teacher chat, content processing, and code generation. Mock API locally with static responses for MVP. Prompt examples: “Generate Angular Element for lesson viewer with tenant ID input using AWS Cognito auth” or “Write NestJS endpoint with RLS for tenant-isolated lesson CRUD with approval status on AWS RDS.”
- **Security**: Validate n8n workflows (e.g., restrict nodes, enforce input/output schemas). Sanitize uploads, secure APIs with Cognito JWT and CORS for enterprise domains, comply with GDPR/CCPA. Use AWS IAM for S3 access and AWS Secrets Manager for tenant-specific Grok API keys.

## MVP Goals
- **Objective**: Build a skeleton app proving end-to-end flows:
  - Student: Browse mock approved lesson, view lesson with Pixi.js interaction (e.g., draggable sprite) and mock Grok-powered AI chat.
  - Lesson-Builder: Create lesson JSON with stages, data links, interaction type, optional n8n workflow, Grok AI teacher prompts, and submit for approval.
  - Interaction-Builder: Create interaction type JSON (Pixi.js config), upload app owner’s n8n workflow JSON, and submit for approval.
  - Content Processing: Upload mock file → Submit for approval → Trigger n8n webhook with approved workflow → Return processed JSON (using mock Grok API).
  - Enterprise Prep: Implement basic multi-tenancy (e.g., `tenant_id` in DB tables) and a proof-of-concept Angular Element for lesson viewing with Cognito auth, showing only approved content.
- **Key Deliverables**:
  - Lesson JSON rendering in frontend with Pixi.js interaction and Grok AI teacher prompts (approved only).
  - Interaction type JSON with mock approved n8n workflow feeding data to Pixi.js.
  - n8n flow triggered by webhook, returning mock processed data.
  - Basic auth (mock Cognito JWT with tenant ID and admin role), Grok token tracking (console logs), commission metrics (mock DB queries for approved content), tenant-aware data queries.
  - Cross-platform: Works in browser and iOS TestFlight, with Angular Elements proof-of-concept.
- **Testing**: Unit tests (Jest for frontend, Mocha/Chai for backend) for JSON parsing, API endpoints, workflow validation, tenant isolation, and approval status handling. E2E tests (Cypress) for login → lesson submission → approval → interaction flow.
- **Modularity**: Develop components separately (e.g., frontend with mock API, backend with Postman). Use MSW for frontend API mocking.
- **Deployment**: Coolify deploys from GitHub to local Docker. Test services communicate (e.g., frontend → API → n8n).

## Constraints
- **Non-Experienced Coder**: Prioritize simple, AI-generated code with clear structure. Use Ionic/Angular and NestJS for beginner-friendly conventions.
- **JSON-Centric**: Lessons, interaction types, n8n workflows, and Grok prompts are JSON, validated with schemas to ensure compatibility.
- **Local-First**: All services run in Docker locally, movable to AWS later.
- **Cross-Platform**: Ionic ensures browser and native builds (iOS/Android). Angular Elements for enterprise web embedding. TestFlight for iOS testing.
- **Security**: Validate all user inputs (e.g., uploaded files, workflow JSON). Limit n8n workflows to safe nodes (e.g., no unauthorized HTTP calls). Use tenant-aware RLS for enterprise isolation. Ensure only approved content is accessible.
- **Performance**: Lazy-load lessons, cache processed data in Redis, offload heavy tasks (e.g., Grok API calls) to backend. Plan for low-latency STT/TTS (future feature) with streaming WebSockets.

## Critical Analysis Requirement
- **Instruction**: For every prompt, critically evaluate the requested changes or features before implementing. If a request may not work, could cause issues (e.g., performance, security, compatibility, scalability, or maintainability problems), or deviates from the app’s architecture, goals, or constraints, do not blindly implement it. Instead:
  - Identify potential issues (e.g., “This approach may overload n8n with complex workflows” or “This feature conflicts with Ionic’s mobile rendering”).
  - Explain why it’s problematic, referencing the app’s structure, MVP goals, or technical constraints.
  - Suggest a better alternative that aligns with the Ionic/Angular, Angular Elements, NestJS, PostgreSQL, n8n, Pixi.js, Redis, AWS Cognito, S3, and Grok API stack, prioritizes simplicity for a non-experienced coder, and maintains modularity, security, approval processes, and cross-platform compatibility.
  - If the request is sound, confirm why it fits and proceed with implementation, ensuring code includes tests and adheres to JSON-centric design with approval status handling.

## Specific Notes
- **n8n Workflows**: For the MVP, the app owner creates initial n8n workflows offline using n8n’s native UI (e.g., localhost:5678) and uploads JSON via the Interaction-Builder interface (simple file input) for approval. Workflows are stored in PostgreSQL (JSONB) with fields: `id`, `interaction_type_id`, `workflow_json`, `created_by`, `input_format`, `output_format`, `tenant_id`, `status` (pending/approved/rejected). They are triggered via n8n API/webhooks only if approved, validated for security (e.g., allowed nodes only).
- **Pixi.js Interactions**: Run in sandboxed environment. JSON defines inputs (e.g., processed data from n8n) and events (e.g., drag end triggers Grok AI chat). Tenant-specific and approval-required for enterprise clients.
- **AI Teacher Prompts**: Stored in lesson JSON (JSONB in PostgreSQL), managed via the Lesson-Builder interface with a dedicated section for creating/editing prompts to guide Grok-powered AI teacher interactions. Require approval before use.
- **LLM Integration**: Use xAI’s Grok API (via https://x.ai/api) for AI teacher chat, content processing (e.g., summarization in n8n), and coding assistance (e.g., Pixi.js code generation). Mock API locally with static responses for MVP. Store tenant-specific API keys in AWS Secrets Manager.
- **Database**: PostgreSQL for relational data (users, lessons, workflows, usage metrics, prompts). JSONB for flexible storage. Supports multi-tenancy via Row-Level Security (RLS) with `tenant_id` for enterprise clients. Hosted on AWS RDS. Pinecone for semantic search (optional for MVP). Include `status` (pending/approved/rejected) for lessons, interaction types, and workflows.
- **Token Tracking**: Log Grok API usage in middleware, store in DB with tenant isolation, push warnings via WebSockets (tenant-namespaced rooms).
- **Commissions**: Track usage (lesson views, interaction runs, workflow executions) of approved content only in DB with tenant isolation. Aggregate for Stripe Connect payouts.
- **Angular Elements**: Package key frontend components (e.g., Lesson View, Lesson Browser) as web components for enterprise clients to embed via a JavaScript blob (e.g., `<script src="platform.js">` and `<lesson-viewer tenant="client-abc">`). Ensure tenant-specific data pulls via API with secure auth (Cognito JWT with tenant claims) and CORS restrictions, showing only approved content.
- **AWS Integration**: Use AWS SDK in NestJS for S3 operations and Cognito for auth. Local mocks: MinIO for S3, mock JWT for Cognito, static responses for Grok API.
- **Approval Process**: Implement submission and approval workflows for lessons, interaction types, and n8n workflows. Store statuses in PostgreSQL (`status` ENUM). For MVP, the app owner manually approves via DB updates; post-MVP, add tenant-specific admin UI for approvals. Use Socket.io for status update notifications.

All prompts should adhere to this context, ensuring code, tests, and workflows align with the MVP goals, technical stack, and modularity. Generate code for Ionic/Angular, Angular Elements, NestJS, PostgreSQL, n8n, Pixi.js, Redis, AWS Cognito, S3, and Grok API unless specified otherwise. Include unit tests with Jest for frontend and Mocha/Chai for backend, and prioritize simplicity for a non-experienced coder. Critically analyze requests, flag issues, and propose better approaches when needed, considering multi-tenancy, approval processes, and AWS/Grok compatibility.