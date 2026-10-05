# Synergy Blossom

**The enterprise agent platform of Synergy Ecosys, built and evolved by Blossom Factory.**

Synergy Blossom will organize company knowledge, business processes, human decisions, integrations, and AI agents in one governed platform. Synergy's internal operations are the first deployment. Client organizations will later use the same platform through isolated workspaces, their own data, and explicitly authorized integrations.

Blossom Factory is the engineering environment derived from ECC that builds this product. Its planner receives bounded requirements, produces implementation plans, and coordinates engineering, review, testing, and evaluation. Production business agents operate independently of the Factory.

**Status:** proposed architecture and implementation requirements, prepared **5 October 2026**. This document specifies work to build. Directories, APIs, commands, agents, and operational targets below are planned contracts; their presence here does not mean they are implemented or tested.

**First product release:** authenticated staff can maintain a brand brief and verified calendar, generate twelve monthly narrative briefs, expand a selected month into text drafts, edit immutable revisions, obtain review, and export Markdown/JSON. Research, creative assets, Instagram publication, additional business domains, and external client rollout have separate gates.

**For the ECC planner:** read sections 1–10 for invariants, sections 11–17 for product contracts, and sections 18–22 for delivery instructions. Plan one work package at a time using its dependencies and acceptance criteria. Preserve the complete architecture while delivering small working slices.

## Contents

1. [Vision, source authority, and scope](#1-vision-source-authority-and-scope)
2. [Responsibilities of Blossom, ECC, LangGraph, LangChain, OpenClaw, and MCP](#2-responsibilities-of-blossom-ecc-langgraph-langchain-openclaw-and-mcp)
3. [Architecture and deployment boundaries](#3-architecture-and-deployment-boundaries)
4. [Complete technology stack](#4-complete-technology-stack)
5. [Repository structure and dependency rules](#5-repository-structure-and-dependency-rules)
6. [Shared platform modules and business domains](#6-shared-platform-modules-and-business-domains)
7. [Organizations, identity, authorization, and tenant isolation](#7-organizations-identity-authorization-and-tenant-isolation)
8. [Database, ingestion, knowledge, and memory](#8-database-ingestion-knowledge-and-memory)
9. [Execution, queues, checkpoints, approvals, and recovery](#9-execution-queues-checkpoints-approvals-and-recovery)
10. [Production agent, workflow, and tool contracts](#10-production-agent-workflow-and-tool-contracts)
11. [Frontend experience and route catalog](#11-frontend-experience-and-route-catalog)
12. [Shared backend API](#12-shared-backend-api)
13. [Marketing: Time, Value, and the first application](#13-marketing-time-value-and-the-first-application)
14. [Marketing backend API](#14-marketing-backend-api)
15. [Integrations, MCP, and optional OpenClaw](#15-integrations-mcp-and-optional-openclaw)
16. [Infrastructure, environments, configuration, and commands](#16-infrastructure-environments-configuration-and-commands)
17. [Growth, observability, costs, and operations](#17-growth-observability-costs-and-operations)
18. [Blossom Factory engineering system](#18-blossom-factory-engineering-system)
19. [Implementation phases and assignable work packages](#19-implementation-phases-and-assignable-work-packages)
20. [Evaluation, verification, and release gates](#20-evaluation-verification-and-release-gates)
21. [Planner handoff templates and copyable prompts](#21-planner-handoff-templates-and-copyable-prompts)
22. [First build sequence and requirements traceability](#22-first-build-sequence-and-requirements-traceability)
23. [Decisions, provenance, and technical references](#23-decisions-provenance-and-technical-references)

---

## 1. Vision, source authority, and scope

### 1.1 Source authority

This README reconciles two supplied documents:

| Source | Authority in this blueprint | Treatment |
|---|---|---|
| First attachment, `Pasted text.txt` | Draft enterprise architecture and Factory/runtime responsibility separation | Retain the distinction between engineering agents and production agents; complete the unfinished repository structure |
| Second attachment, `Pasted text (2).txt` | v0 system requirements, especially Marketing, platform ownership, APIs, and phased delivery | Preserve the first text release, Time/Value contracts, tenancy, revisions, approvals, queue recovery, and later Marketing milestones |
| This README | Proposed consolidated implementation baseline for the Factory | Adds demand intake, explicit agent/workflow contracts, Factory delivery rules, data onboarding, and an actionable platform backlog |

Use **Synergy Blossom** as the platform name and **Blossom Factory** as the engineering system name. The v0's **Synergy Bloom** refers to the same platform; it is not another product or tenant. Use `synergy-blossom`, the package prefix `@blossom/`, and `/api/blossom` for new implementation. If existing code already uses Bloom identifiers, document and migrate them deliberately.

The original company marketing artifact has not been supplied as readable content. The platform must not assume its brand facts, processes, or approved claims. Use the supplied v0 as the requirements baseline and obtain actual pilot business records in M0.

### 1.2 Product model

One platform supports many organizations, business domains, and approved processes. Each organization has its own people, policies, knowledge, integrations, agent deployments, usage limits, and audit trail.

| Concept | Meaning |
|---|---|
| Organization / tenant / workspace | One isolated operating environment. `tenantId` and `workspaceId` represent the same boundary in v1 |
| Customer account | A commercial relationship managed by Synergy; it does not itself grant access to the customer's tenant |
| Organizational unit | Optional department/team inside a tenant; narrower permissions, never an independent implicit tenant |
| Domain | Business responsibility such as Marketing, Contracts, Finance, Sales, or Operations |
| Capability / skill | A typed, reusable operation with inputs, outputs, authorization, and ownership |
| Agent definition | Versioned code, prompts, schemas, permitted tools, and evaluation requirements |
| Agent deployment | A tenant's enabled configuration of an approved agent version, including policy and budget |
| Workflow definition | An approved, versioned process coordinating deterministic steps, agents, and human tasks |
| Demand | A recorded business request linked to requester, scope, objective, inputs, execution, and outcome |
| Run | One execution of a known task/workflow against an immutable input snapshot |
| Revision | An immutable version of an input or output subject to review and dependency tracking |

### 1.3 Delivery scope

| Stage | Required business outcome | Explicitly later work |
|---|---|---|
| Local proof: M0–M2 | One brand/month produces useful, inspectable text | Shared tenancy, annual generation, production operations |
| Staff text release: P0–P3 + M0–M4 | Approved inputs → yearly outline → selected-month drafts → review → export | Live research, imagery, publishing, payments, legal signatures |
| Durable orchestration: P5 + M5 | Versioned LangGraph workflows recover, pause, resume, and selectively regenerate | Arbitrary user-authored executable workflows |
| Client readiness: P4 | Controlled onboarding, isolation, delegation, data governance, quotas, and tested recovery | Unlimited scale or unsupported capacity claims |
| Advanced Marketing: M6–M9 | Reviewed research, creative assets, approved publication, measurement | Unreviewed automatic approval or model retraining |
| New domains: P6 | Domain discovery followed by one controlled business workflow per domain | Claiming complete ERP, CRM, accounting, HR, or legal systems before requirements exist |

The platform should eventually absorb diverse internal and client demands through a process catalog and integration contracts. It cannot reliably automate an unspecified business process. Each new demand type requires an owner, a supported contract, relevant data, permissions, acceptance criteria, and a controlled release.

### 1.4 Architectural decisions

1. Use one TypeScript monorepo, a modular NestJS API, and separate worker processes.
2. Keep organization/tenant governance in Blossom's platform kernel.
3. Use LangGraph as the target engine for stateful business workflows; ordinary functions are sufficient for the first Marketing proof.
4. Use LangChain components for models, tools, structured outputs, and retrieval where useful.
5. Use ECC as the foundation/reference for Blossom Factory's engineering workflows.
6. Introduce OpenClaw only for a concrete channel/operator use case.
7. Use PostgreSQL as durable truth; Redis delivers work and coordinates temporary activity.
8. Reuse platform services across domains; give each domain its own business rules, permissions, and table ownership.
9. Keep every external consequential action behind application policy and a registered capability.
10. Start with approved brand/calendar data. Add semantic retrieval, more frameworks, or service extraction when a measured workload justifies them.

## 2. Responsibilities of Blossom, ECC, LangGraph, LangChain, OpenClaw, and MCP

### 2.1 Responsibility matrix

| Component | Primary job | Role in this system | Does not own |
|---|---|---|---|
| **Synergy Blossom** | Enterprise product and governance | Tenants, users, demands, domain data, policies, agent deployments, approvals, integrations, audit, and usage | The coding harness's implementation |
| **Blossom Factory** | Build and evolve Blossom software | Convert bounded requirements into plans, implementation, tests, evaluations, and reviewable releases | Production customer processes or live business approvals |
| **ECC** | Reusable engineering agents, skills, rules, hooks, commands, and verification patterns | Base/reference for the Factory's engineering behavior | Blossom's business database, SaaS tenancy, or production orchestration |
| **Coding harness** | Execute engineering turns and provide developer tools | Claude Code, Codex, Cursor, or another supported environment loads the applicable Factory adapter | Production tenant permissions |
| **LangGraph** | Stateful orchestration and execution | Explicit graphs, checkpoints, controlled branching, interrupts, and business workflow recovery | User identity, tenant isolation, billing, or the meaning of an approval |
| **LangChain** | AI components and higher-level agent abstractions | Model/provider adapters, tool definitions, structured generation, retrieval, optional bounded agent loops | Enterprise governance or a second independent scheduler |
| **OpenClaw** | Agent host, sessions, channels, and operator environment | Optional channel bridge or isolated operator integration | Shared-client isolation, Blossom's canonical data, or approval policy |
| **MCP** | Standard interface for AI applications to tools/data | Optional standardized connector surface exposing approved tools, resources, and prompts | Workflow execution, business authorization, or automatic trust |
| **Model provider / LLM** | Generation, interpretation, or bounded decisions | Produces candidate outputs within application-controlled context and budgets | Factual authority, access grants, database writes, or release decisions |
| **BullMQ** | Queue jobs and schedule delivery | Worker admission/delivery/retries at the execution boundary | LangGraph state or authoritative business outcome |

LangChain's higher-level agents are built on LangGraph; LangGraph can also be used independently. For Blossom, the preferred advanced pattern is an explicit LangGraph workflow containing deterministic domain operations and LangChain-powered model nodes. Avoid wrapping the same task in several uncontrolled agent loops. See the official [LangChain overview](https://docs.langchain.com/oss/javascript/langchain/overview) and [LangGraph overview](https://docs.langchain.com/oss/javascript/langgraph/overview).

### 2.2 Execution ownership

| Boundary | Owner | Requirement |
|---|---|---|
| Plan and implement code | Factory + coding harness | Engineering permissions, isolated branch/worktree, CI and reviewer evidence |
| Admit a business request | Blossom application services | Validate task, actor, tenant, inputs, permissions, budget, and idempotency |
| Deliver a run to a worker | Outbox dispatcher + BullMQ | Recoverable delivery; PostgreSQL retains the job identity |
| Execute workflow steps | Simple runtime initially; LangGraph from M5 | Registered definition/version; durable results; bounded retries |
| Call a model | Model gateway using LangChain adapters | Provider limits, output schema, token/cost accounting, cancellation/deadline |
| Call a tool or integration | Blossom capability gateway | Current authorization, connection ownership, policy, approval, and receipt handling |
| Accept a human decision | Blossom approvals service | Exact subject revision, reviewer authority, current eligibility, audit |
| Receive a channel message | Optional OpenClaw adapter | Resolve sender to a Blossom identity; unknown/ambiguous identities receive no business access |

Blossom's core API and workers must start and continue operating without ECC or OpenClaw installed. Factory prompts, developer memory, and coding commands must not become production runtime dependencies.

### 2.3 Runtime and harness terminology

A runtime executes an agent/workflow. A harness surrounds a model with instructions, context, tools, execution behavior, and session handling. Projects use these terms differently. Always specify the owner of state, permissions, recovery, and external effects instead of relying on the label alone.

Factory engineering agents and production business agents are separate identities with separate credentials, memory, tool permissions, and release procedures. A Factory security review is not approval to publish a client's content or modify its ERP.

## 3. Architecture and deployment boundaries

### 3.1 Product architecture

Dashed edges represent later optional paths. Logical services initially share packages and a PostgreSQL application database; they do not each require a microservice.

```mermaid
flowchart TD
    subgraph Experience["Experience"]
        Web["Next.js web app"]
        Clients["Scoped client API consumers"]
        Channels["Optional OpenClaw bridge"]
    end
    subgraph Kernel["Blossom platform kernel"]
        API["NestJS API"]
        Governance["Identity, tenants, policies, approvals"]
        Domains["Demand intake and domain services"]
        Admission["Run registry, budgets, outbox"]
        API --> Governance
        API --> Domains
        Domains --> Admission
    end
    subgraph Execution["Execution processes"]
        Dispatch["Dispatcher and scheduler"]
        Queue["BullMQ queues"]
        AI["AI worker and runtime adapter"]
        Graph["LangGraph workflows and domain agents"]
        Components["LangChain and deterministic code"]
        Integration["Integration and rendering workers"]
        Dispatch --> Queue
        Queue --> AI
        AI --> Graph
        Graph --> Components
        Queue --> Integration
    end
    subgraph Data["Data and connectivity"]
        PG["PostgreSQL records and checkpoints"]
        Redis["Redis queue datastore"]
        Objects["Private S3 documents and assets"]
        Tools["Authorized capability and MCP gateway"]
    end
    subgraph External["External systems"]
        IdP["Keycloak OIDC"]
        LLM["Model provider APIs"]
        Business["ERP, CRM, calendar, Meta APIs"]
    end
    Web -->|"Same-origin proxy"| API
    Clients --> API
    Channels -.-> API
    Governance --> IdP
    Admission --> PG
    Admission --> Dispatch
    Domains --> PG
    Domains --> Objects
    Queue --> Redis
    Graph --> PG
    Components --> LLM
    Components --> Tools
    Integration --> Tools
    Integration --> Objects
    Tools -->|"Application authorization"| Domains
    Tools --> Business
```

OpenTelemetry and redacted logs span the web server, API, dispatcher, workers, model calls, and integrations. Business audit is a separate durable record of consequential changes; it is not replaced by telemetry.

### 3.2 Factory architecture

```mermaid
flowchart TD
    Requirements["README and work package"] --> Plan["ECC planner and architecture review"]
    Plan --> Contracts["Schemas, dependencies, acceptance criteria"]
    Contracts --> WebWork["Frontend implementation"]
    Contracts --> ServerWork["Backend and database implementation"]
    Contracts --> AgentWork["Workflow and integration implementation"]
    WebWork --> Verify["Tests, evaluations, security, code review"]
    ServerWork --> Verify
    AgentWork --> Verify
    Verify -->|"Findings"| Plan
    Verify -->|"Passed gates"| Candidate["Reviewable release candidate"]
    Candidate --> Release["Controlled product release"]
    Release --> Feedback["Sanitized operational feedback"]
    Feedback --> Plan
```

### 3.3 Deployable units

| Unit | Responsibilities | Scaling boundary |
|---|---|---|
| `apps/web` | UI, server rendering, fixed-target API proxy | Web traffic |
| `apps/api` | Identity/session endpoints, REST API, command admission, domain services | Short HTTP requests |
| `apps/workers/ai` | Model calls and registered workflow execution | Provider throughput and run concurrency |
| `apps/workers/outbox` | Deliver committed work/events; reconcile incomplete delivery | Outbox backlog |
| `apps/workers/scheduler` | Materialize due schedules into idempotent run commands | Number of due occurrences |
| `apps/workers/integrations` | Imports, sync, webhooks, external actions, reconciliation | Connector-specific quotas |
| `apps/workers/rendering` | Isolated deterministic media/export rendering | CPU, memory, and browser capacity |
| Keycloak | Account authentication, MFA, federation | Identity-provider operations |
| PostgreSQL, Redis, S3 | Durable records, delivery coordination, binaries | Independent capacity and recovery policies |
| Optional OpenClaw host | Channel/operator sessions per trust boundary | Isolated deployment; not part of the base release |

The API does not hold open a request for an entire year of generation. Rendering cannot exhaust the AI pool. Workers use the same authorized domain services through package interfaces; extraction to internal APIs can happen later without moving business rules into prompts.

## 4. Complete technology stack

### 4.1 Baseline stack

Choose compatible stable versions during P0, pin exact versions in the lockfile, pin container tags/digests, and record them in `docs/architecture/versions.md`. Node.js **24 LTS** is the baseline runtime. No unverified claim about the latest version or benchmark is part of this plan.

| Layer | Selected technology | Responsibility | Introduced |
|---|---|---|---|
| Language | TypeScript, strict mode | Web, backend, domain logic, workflows, tooling, contracts | P0/M0 |
| Runtime | Node.js 24 LTS; `tsx` for local TS tools | API/workers/CLI execution | P0/M0 |
| Monorepo | pnpm workspaces | Explicit packages, scripts, dependency graph, lockfile | P0 |
| Frontend | Next.js App Router + React | Application shell, routes, forms, review, server rendering | M2 |
| Styling | Tailwind CSS | Theme tokens and layout | M2 |
| Components | shadcn/ui | Owned component source in `packages/ui` | M2 |
| Forms | React Hook Form + Zod | Structured forms and field errors | M2 |
| Client data | TanStack Query | Tenant-scoped server state, mutations, polling | M3 |
| API | NestJS REST | Dependency injection, controllers, policies, domain composition | M2/P1 |
| API contract | `@nestjs/swagger` OpenAPI; `openapi-typescript` | Document implemented routes and generate client types | P1 |
| Runtime validation | Zod | Request, event, task, tool, and model-output validation | M0/P0 |
| Identity | Keycloak OIDC + `openid-client` | Authorization-code flow with PKCE; external account/MFA management | P1 |
| Sessions | Opaque server sessions in PostgreSQL | Revocation, expiry, secure HTTP-only browser cookie | P1 |
| Authorization | Blossom RBAC + explicit resource/policy checks | Human/service permissions and tenant/delegation enforcement | P1 |
| Database | PostgreSQL | Business records, revisions, runs, jobs, approvals, audit | P1 |
| Database access | Drizzle ORM + Drizzle Kit + `pg` | Queries, repositories, reviewed SQL migrations, bounded pools | P1 |
| Queue datastore | Redis locally; verified compatible managed service in production | Persistent BullMQ data; dedicated no-eviction queue instance | P2 |
| Job framework | BullMQ + `@nestjs/bullmq` | Delivery, job attempts, delayed work, workers | P2 |
| Scheduling | Blossom scheduler backed by PostgreSQL + Luxon | Durable schedules, timezone handling, due occurrences | P5/M5 |
| AI components | `@langchain/core`, `langchain`, approved provider adapters | Models, structured output, tools, optional agent loops | M0; broader use in M5 |
| Initial model adapter | `@langchain/openai` | Initial provider integration; model selected by evaluation | M0 |
| Workflow engine | `@langchain/langgraph` | Stateful graphs and bounded specialist collaboration | P5/M5 |
| Checkpoints | PostgreSQL-backed LangGraph persistence behind a tenant-aware adapter | Authorized resume, state inspection, retention | P5/M5 |
| MCP | Official TypeScript SDK compatible with the selected protocol release | Client/server connector adapters behind policy checks | P4/P5 |
| Dates | Luxon + verified event records | Local dates, IANA timezones, campaign windows | M1 |
| Storage | Private Amazon S3 + AWS SDK v3 | Documents, attachments, export artifacts, media | M6/M7; earlier if imports require files |
| Cryptography/secrets | Node crypto, host secret store, versioned encryption keys | Encrypt OAuth credentials; hash API keys; rotate keys | P1/P4 |
| Rendering | Approved React/HTML/CSS templates + Playwright | Deterministic text layout and image/export checks | M7 |
| Publication | Official Meta Instagram API adapter | Approved professional-account publication and receipts | M8 |
| Containers | Docker, Docker Compose for local services | Reproducible API/workers/Keycloak and development dependencies | P1/P2 |
| Web hosting | Vercel | Deploy the web application | P3 |
| API/worker hosting | Render Docker services + background workers | Conventional API and continuously running processes | P3 |
| Database hosting | Managed Render PostgreSQL | Backups and controlled application storage | P3 |
| Queue hosting | Render Key Value or compatible managed Redis | Verify BullMQ compatibility, persistence, no-eviction policy | P3 |
| Logging | Pino + NestJS integration | Structured, redacted logs | P2 |
| Traces/metrics | OpenTelemetry SDK + Collector | Correlate API, jobs, tools, and model operations | P2/P3 |
| Telemetry visualization | Grafana; Prometheus/Loki/Tempo or managed equivalents | Operational dashboards; exact hosting chosen in P3 | P3/P4 |
| Unit/integration testing | Jest, `@nestjs/testing`, Supertest | Domain rules, authorization, API, execution failure paths | M1/P1 |
| Browser tests | Playwright Test | Critical authenticated business journeys | M3/M4 |
| Load tests | k6 | Workload, admission, fairness, backlog, recovery | P4 |
| Engineering quality | ESLint + Prettier + TypeScript checks | Code consistency and import boundaries | P0 |
| CI/CD | GitHub Actions | Checks, evaluated release artifacts, migrations, controlled deployments | P1/P3 |
| Engineering system | ECC-derived Factory + selected coding-harness adapters | Planning, implementation, review, test/evaluation workflows | F0/F1 |
| Documentation | Markdown + Mermaid + ADRs | Architecture, contracts, requirements, decisions, runbooks | P0/F0 |

P0 must prove the chosen ESM/module configuration can load NestJS, LangChain packages, worker entry points, and the test runner. Commit that configuration instead of discovering incompatible imports during deployment. Zod schemas and generated OpenAPI types must agree; TypeScript types alone do not validate runtime input.

### 4.2 Conditional additions

| Technology/capability | Trigger | Integration boundary |
|---|---|---|
| OpenClaw | An approved operator/channel use case | Isolated bridge/host calling scoped Blossom commands |
| LangSmith | Approved need for AI traces/evaluations | Optional export under tenant and retention policy |
| PostgreSQL `pgvector` | Retrieval quality/scale exceeds explicit approved-record lookup | Derived tenant-scoped index; never evidence authority |
| Deep Agents | Measured need for planning/filesystem/subagent harness features | A bounded runtime implementation; not a second platform kernel |
| Python analytics service | A specific ML/data library or processing requirement | Typed API/job; owns no implicit tenant access |
| Remotion | Approved video production requirement | Separate render pool and media contract |
| Terraform | Infrastructure provisioning requires reproducible automation | Infrastructure repository/package, no business logic |
| Read replicas, partitioning, dedicated tenant databases | Measured volume or contractual isolation requirement | Storage routing hidden behind repositories; consistent authorization |
| Payment processor | Validated SaaS subscription/charging requirements | Billing adapter; distinct from Finance accounting |
| Kubernetes / event-stream platform | Measured deployment or throughput requirements | Later infrastructure ADR and operational ownership |

Open-source framework use does not eliminate provider, hosting, support, or engineering costs. Verify current licenses for pinned dependencies. Do not assign a license to new Blossom product code merely by inheriting ECC files.

## 5. Repository structure and dependency rules

### 5.1 Target monorepo

Reserve later boundaries in the documentation; create directories when their package starts. The root `AGENTS.md` supplies engineering instructions to coding agents. It is not a production agent specification or an LLM business workflow.

```text
synergy-blossom/
  README.md
  AGENTS.md
  package.json
  pnpm-workspace.yaml
  pnpm-lock.yaml
  .env.example
  .node-version
  .github/
    workflows/                         CI, evaluated releases, migrations
    ISSUE_TEMPLATE/                    Work-package issue template
  .claude/                             Harness adapter; only if selected
  .codex/                              Harness adapter; only if selected
  .cursor/                             Harness adapter; only if selected
  blossom-factory/                      Engineering assets; excluded from product builds
    README.md
    upstream/                          ECC provenance, commit, license, update notes
    agents/                            Verified stock roles and proposed custom roles
    skills/                            Blossom architecture and engineering procedures
    rules/                             Tenancy, data, APIs, runtime, security, review rules
    commands/                          Planner/implementation/review entry points
    hooks/                             Harness-specific validated automation
    adapters/                          Claude Code, Codex, Cursor configuration/mapping
    templates/                         Requirement, plan, PR, ADR, release templates
    memory/                            Sanitized engineering decisions, not client records
    evaluations/                       Factory task and review effectiveness
  apps/
    prototype/
      src/cli.ts                       Local monthly narrative proof
    web/
      src/app/                         Next.js routes and API proxy
      src/features/                    Marketing, demands, review, workspace UI
      src/lib/                         Typed API client and session-aware query keys
    api/
      src/main.ts
      src/app.module.ts
      src/modules/                     HTTP/controller and DI composition
        identity/
        workspaces/
        demands/
        catalog/
        knowledge/
        runtime/
        approvals/
        integrations/
        marketing/
    workers/
      src/ai.ts
      src/outbox.ts
      src/scheduler.ts
      src/integrations.ts
      src/rendering.ts
  packages/
    primitives/                        IDs, money/date values, domain errors, public ports
    api-contracts/                     Public validated requests/results, generated types
    events/                            Versioned integration/domain event schemas
    platform/
      src/identity/
      src/workspaces/
      src/policies/
      src/demands/
      src/registry/
      src/catalog/
      src/knowledge/
      src/documents/
      src/calendar/
      src/execution/
      src/approvals/
      src/usage/
      src/audit/
    agent-runtime/
      src/contracts/                   Runtime, model, tool, and workflow interfaces
      src/simple/                      Application-controlled initial runtime
      src/langgraph/                   Graph adapter and authorized persistence
      src/models/                      Provider selection, validation, budgets
      src/tools/                       Authorized capability invocation gateway
    agents/                            Production definitions; separate from Factory roles
      src/marketing/
        definition.ts
        graph.ts                       Introduced in M5
        state.ts
        prompts/
        tools/
        policies/
      src/contracts/                   Later approved contract agent
      src/finance/                     Later approved finance agent
      src/operations/                  Later approved operations agent
    marketing/
      src/domain/                      Profiles, claims, plans, revisions, policies
      src/application/                 Use cases and orchestration-independent operations
      src/ports/                       Public service/repository interfaces
      src/skills/time/
      src/skills/value/
      src/research/
      src/creative/
      src/publishing/
      src/analytics/
    legal-contracts/                   Future business domain, not API type definitions
    finance/                           Future deterministic finance domain
    operations/                        Future operational domain
    database/
      src/schema/                      Owned SQL schemas and tables
      src/repositories/
      src/transactions/                Transaction-local trusted tenant context
      migrations/                      SQL migrations and RLS policies
      seeds/                           Sanitized local fixtures
    integrations/
      src/identity/
      src/storage/
      src/research/
      src/mcp/
      src/instagram/
      src/openclaw/                    Optional scoped adapter
      src/crm/                         Later customer-specific adapters
      src/erp/                         Later customer-specific adapters
    ui/                                Owned shadcn/ui components and design tokens
    config/                            Shared TS, lint, test, environment schemas
  examples/                            Synthetic brand, calendar, demand, output fixtures
  evaluations/
    marketing/                         Cases, rubrics, manifests, results
    platform/                          Tool policy, routing, tenant isolation cases
  tests/
    integration/                       API, SQL, queues, recovery, checkpoints
    e2e/                               User journeys
    load/                              k6 workloads
  infrastructure/
    docker/                            Dockerfiles and compose.dev.yml
    render/                            Service and worker deployment definitions
    identity/                          Keycloak dev realm/configuration
    observability/                     Collector/dashboard configuration
  docs/
    architecture/                      Models, diagrams, versions, deployment topology
    requirements/                      Versioned business specifications
    contracts/                         API, events, tasks, tool schemas
    adr/                               Architecture decision records
    backlog/                           Work packages, plans, dependency map, evidence
    runbooks/                          Restore, rotation, recovery, incident procedures
```

### 5.2 Import and ownership rules

| Package/boundary | May depend on | Must avoid |
|---|---|---|
| Web | UI, public API contracts, browser-safe primitives | Database, server secrets, Factory files, private repositories |
| Domain logic | Primitives and explicit public ports | NestJS controllers, queue clients, provider SDKs, other domains' private tables |
| Platform kernel | Primitives, its own application contracts | Importing Marketing/Finance/Contracts implementations |
| Agent definitions | Runtime contracts, public domain service ports, validated schemas | Direct SQL, raw OAuth credentials, bypassing approval services |
| Runtime | Public task/tool ports and provider/persistence adapters | Domain-specific pricing/legal decisions hidden in generic code |
| Database/integration adapters | Domain ports and relevant SDKs | Becoming a second owner of business rules |
| API/workers | Required packages through DI composition | Moving canonical rules into controllers or duplicating them per entry point |
| Factory | Specifications and engineering repository tooling | Runtime import by any product package; ingestion of live client memory |

Domain packages expose public commands/queries and events. Table migrations are centralized in `packages/database`, but table/business-rule ownership remains explicit. Shared approval subjects and workflow definitions are registered through DI, so the platform does not import every domain to discover them.

Start the Factory inside the monorepo for convenience. It can later be extracted to a separate engineering repository without affecting deployed business agents. Preserve ECC attribution and provenance; verify installed harness discovery paths instead of assuming `blossom-factory/agents` loads automatically.

## 6. Shared platform modules and business domains

### 6.1 Platform kernel

| Module | Owns | Initial implementation | Growth behavior |
|---|---|---|---|
| Identity | Local identity references, sessions, service accounts | Keycloak-backed staff access | Client federation/SSO and scoped API credentials |
| Workspaces | Tenants, members, delegations, optional organizational units | Internal workspace plus synthetic isolation fixtures | Client onboarding and support-access lifecycle |
| Policy | Resource/action/field permissions, approved configurations | RBAC and explicit object checks | Contextual policies without changing tenant boundaries |
| Demand intake | Request objective, type, scope, owner, outcome | Structured request tracking linked to Marketing runs | Process catalog, qualification, routing, human triage |
| Agent/workflow registry | Definition versions, deployment configuration, supported task types | Code-defined Marketing definition | Tenant-enabled approved domain workflows |
| Customer registry | Commercial company/contact references | Approved client/account metadata | CRM adapters; no implicit cross-tenant content access |
| Offer catalog | Versioned products/services, prices, validity, availability | Marketing input offers | Shared approved commercial terms for other domains |
| Calendar | Event definitions, verified occurrences, date/timezone utilities | Curated local/tenant calendar | Dedicated campaign calendars, deadlines, schedules |
| Knowledge/evidence | Sources, applicability, versions, review, allowed use | Structured approved factual records | Document ingestion, retrieval, lineage, semantic index |
| Documents/assets | Ownership, object metadata, hashes, versions | Metadata; files when needed | S3 ingestion, creative media, export artifacts |
| Execution | Runs, jobs, leases, outbox/inbox, cancellation, scheduler | Short controlled sequence in workers | Durable graphs and bounded child runs |
| Approvals/human tasks | Exact revision, reviewer, decision, policy evidence | Claims, offers/evidence, content review | Domain-specific review policies and approval chains |
| Integrations | Connections, secrets references, cursors, webhook deliveries | Selected allowlisted adapters | Client ERP/CRM and MCP connectivity |
| Usage/entitlements | Limits, reservations, actual usage, cost attribution | AI run and tenant caps | Commercial plans; billing adapter if required |
| Audit/operations | Durable business events, safe diagnostics, operational controls | Traceable edits, runs, approvals | Client support, recovery, incident controls |

Each future domain consumes these services rather than rebuilding identity, evidence, queues, approvals, or secrets management.

### 6.2 Demand intake and routing

A demand contains `tenantId`, requester, optional organizational unit/brand, task type, objective, structured payload or authorized attachment references, sensitivity, deadline, priority, and acceptance description. The server adds IDs, authorization context, audit, and policy version.

Proposed demand states: `submitted`, `needs_information`, `qualified`, `queued`, `in_progress`, `waiting_for_human`, `resolved`, `rejected`, `cancelled`. A demand may produce multiple runs. Run success alone does not establish that the business objective has been accepted; resolution stores outcome references and the required owner decision.

Routing rules:

1. Determine the tenant and requester from verified identity.
2. Validate a supported task contract and check the tenant's enabled process catalog.
3. Require missing inputs or route unsupported requests to human triage.
4. Resolve authorized data and applicable approval/cost policy.
5. Create the immutable run snapshot and idempotent dispatch command.
6. Link results and required human tasks back to the demand.

A model may suggest a task category or summarize a request. Its suggestion cannot enable a disabled agent, grant permissions, choose another tenant, or execute an unknown tool. In v1, routing is deterministic against the supported catalog; a general autonomous dispatcher is a later evaluated feature.

### 6.3 Domain roadmap

| Domain | Initial controlled workflow candidate | Reusable foundation | Domain-specific requirements still needed |
|---|---|---|---|
| Marketing | Time/Value narrative planning and review | Catalog, evidence, dates, runs, review, assets | Pilot brand, market, rubric, approved claims |
| Contracts | Extract clauses/obligations and prepare a review summary | Documents, evidence, deadlines, review, integration framework | Jurisdiction, clause sources, reviewers, signature authority, retention |
| Finance | Import and reconcile records; prepare a discrepancy report | Connectors, runs, approvals, exact-decimal primitives | Accounting rules, source system, tolerances, reviewer/payment boundaries |
| Operations | Coordinate a known recurring internal process | Demands, schedules, typed tasks, notifications, approvals | Process owner, steps, exceptions, external write permissions |
| Sales/support | Retrieve approved account context and prepare a response/action proposal | Customer registry, evidence, channels, scoped tools | CRM records, consent, response policy, assignment rules |
| Procurement/HR | Later after domain discovery | Shared governance and execution | Sensitive-data policy, obligations, authorized actions, domain evaluation |

These are starting candidates, not completed specifications. Contracts cannot legally sign and Finance cannot move money merely because an agent can produce text. Deterministic calculations, required authority, and external-action policy remain domain-owned.

## 7. Organizations, identity, authorization, and tenant isolation

### 7.1 Authentication

Use Keycloak as the OIDC provider. NestJS performs the authorization-code flow with PKCE using `openid-client`, validates state/nonce/issuer/audience/approved redirects, and creates a revocable opaque application session. Passwords, account recovery, and MFA remain with the identity provider.

The browser uses a same-origin Next.js proxy. The public callback is `/api/blossom/auth/callback`, mapping to NestJS `/v1/auth/callback`. Forward only permitted headers/cookies to a fixed API target. Store refresh tokens encrypted server-side; implement expiry, refresh failure, revocation, and logout. Use HTTP-only, secure, appropriate SameSite cookies, CSRF protection, and origin checks for cookie-authenticated mutations. Production cookies should use an appropriate `__Host-` configuration.

The proxy is transport. NestJS authenticates and authorizes every request independently. Browser code never receives model keys, database credentials, identity client secrets, or OAuth refresh tokens.

### 7.2 Roles and permissions

| Role | Default responsibility | Restriction |
|---|---|---|
| Platform operator | Provision tenants, operate services, configure entitlements | No default client-content access; explicit audited support grant required |
| Workspace administrator | Manage members, policies, enabled agents, connections | Authorized tenant only |
| Domain editor/operator | Maintain inputs and initiate allowed tasks | Cannot approve/publish/pay/sign without the relevant grant |
| Domain approver | Review exact outputs under domain policy | Cannot override invalid evidence or required separation of duties |
| Marketing publisher | Connect authorized accounts and schedule approved packages | Current exact-package approval and channel eligibility required |
| Client reviewer | Inspect assigned work and submit permitted feedback/decisions | No secrets, raw prompts, or unrelated brands/clients |
| Analyst/audit reader | Read defined metrics or permitted business history | Sensitive fields and exports remain independently scoped |
| Service account / agent principal | Execute specified task/tool capabilities | Tenant-bound scopes, expiry, quota, revocation, and deployment identity |

Implement server permissions such as `demands.create`, `marketing.content.write`, `marketing.content.approve`, `marketing.publish.schedule`, `integrations.manage`, and `audit.read`. Permission checks combine actor grants, tenant membership/delegation, resource ownership, agent deployment grants, and current domain policy. An agent never gains more authority than the approved service/delegation and initiating request permit.

Synergy staff receive explicit, expiring, revocable access to assigned client tenants and resources. A platform administrator role is not a blanket ability to query every customer's knowledge. The commercial customer registry must expose only approved fields across workspace boundaries.

### 7.3 Isolation invariants

| Surface | Required isolation |
|---|---|
| SQL | Trusted tenant transaction context, scoped queries, composite tenant-aware relationships, RLS defense in depth |
| Object/field/action access | Resolve nested ownership and permitted fields; no authorization by ID shape alone |
| Jobs and scheduled work | Recheck initiating/service authority, tenant state, policy, and budget at execution |
| Graphs/checkpoints | Resolve owned run/thread and enforce scoped persistence on reads, writes, lists, and resume |
| Retrieval/memory | Scope source selection before keyword/vector search; preserve document/field permissions |
| Redis/cache | Tenant/resource/version-aware keys; queues contain minimal references |
| Browser/Next.js cache | Private user data must not enter shared public caches; clear client state on session/tenant change |
| Objects/downloads | Authorize metadata before issuing short-lived signed URLs; object prefixes are organizational only |
| Integrations | Credential/account/connection ownership resolved server-side; no model-selected arbitrary credential |
| Logs/traces/evaluations | Redaction, authorized visibility, retention; no unapproved cross-client content |
| Factory memory | Engineering-only sanitized decisions and fixtures; no production secrets/client source documents |
| OpenClaw | Separate gateway/credentials/storage per client trust boundary if introduced |

Client-supplied `X-Workspace-Id`, nested IDs, prompts, and queue fields are selectors to validate. They are never evidence of authority. Revoking membership, an agent deployment, or an integration prevents delayed consequential actions under that authority.

## 8. Database, ingestion, knowledge, and memory

### 8.1 Storage ownership

Start with one managed PostgreSQL application database and explicit SQL schemas. Keycloak uses a separate database/role and lifecycle. Product code must not mutate identity-provider private tables.

| Schema/module owner | Principal tables | Important fields/relationships |
|---|---|---|
| `identity` | `users`, `sessions`, `service_accounts`, `api_keys` | OIDC subject/issuer, session expiry/revocation, hashed API credential, scopes |
| `workspaces` | `tenants`, `memberships`, `delegations`, `org_units`, `resource_grants` | Status, locale/timezone, role/policy, expiry, resource scope |
| `crm` | `customer_accounts`, `contacts`, `account_links` | Commercial identity; explicitly allowed relationship to a tenant |
| `demands` | `demands`, `demand_events`, `demand_run_links` | Requester, type, objective, payload version, owner, state, outcomes |
| `registry` | `agent_definitions`, `workflow_definitions`, `agent_deployments` | Immutable version/hash, supported task schemas, enabled policy/budget |
| `catalog` | `offers`, `offer_versions`, `offer_assignments` | Approved benefit/mechanism, exact-decimal price/currency, validity, availability |
| `knowledge` | `sources`, `evidence`, `evidence_versions`, `documents`, `document_versions`, `chunks`, `ingestion_jobs`, `source_dependencies` | Origin, ownership, hash, review/expiry, classification, allowed public use, lineage |
| `calendar` | `event_definitions`, `event_versions`, `event_occurrences` | Market, local date, verified source/version, recurrence, lead/follow-up windows |
| `runtime` | `runs`, `input_snapshots`, `run_steps`, `jobs`, `run_leases`, `outbox`, `inbox`, `schedules`, `schedule_occurrences`, `tool_invocations` | Versions, attempt/state, deadline, fencing token, idempotency hash, effect receipts |
| `approvals` | `review_tasks`, `approval_requests`, `approval_decisions` | Registered subject type, exact revision/hash, policy version, reviewer, decision |
| `integrations` | `connections`, `credential_refs`, `sync_cursors`, `webhook_subscriptions`, `webhook_deliveries`, `external_receipts` | Owner/scopes, key version, external identity, cursor, receipt/reconciliation |
| `usage` | `entitlements`, `budget_reservations`, `usage_records` | Tenant/run/tool/provider, actual units/cost, reservation expiry, rate version |
| `audit` | `audit_events` | Actor/agent, tenant, action, subject/revision, safe change metadata, correlation |
| `marketing` | `brands`, `profile_versions`, `audiences`, `claim_revisions`, `plans`, `plan_revisions`, `content`, `content_revisions`, `signals`, `templates`, `assets`, `publication_packages`, `publishing_jobs`, `metrics`, `feedback` | Brand ownership, immutable inputs/outputs, eligibility, publication and metric provenance |
| `checkpoints` | Adapter-owned checkpoint/state records and thread bindings | Tenant/run/thread ownership, graph/state version, retention; never public raw access |
| `legal_contracts`, `finance`, `operations` | Later domain-owned tables | Defined only after their process/data specifications are approved |

Rows requiring identities, filtering, relations, independent permissions, or lifecycle are normalized. Schema-validated JSONB holds bounded snapshots, settings, generated structures, and graph state. Do not use an unversioned JSON blob as the entire enterprise data model.

### 8.2 Database conventions

- Use server-generated UUIDs, UTC timestamps for instants, and separate local date/timezone fields where business semantics require them.
- Tenant-owned rows carry `tenant_id`; brand-owned rows also carry `brand_id`. Use unique/composite keys such as `(tenant_id, id)` and matching foreign keys to prevent cross-tenant relationships.
- Common indexes begin with tenant and relevant resource, followed by status/date/ID as appropriate. Add indexes from measured query plans.
- Store monetary values as exact decimal/numeric with currency; never floating-point LLM arithmetic.
- Mutable identity records use an optimistic `version`/ETag. Material edits create immutable profile, offer, evidence, plan, content, or package revisions.
- Use separate migration/admin and runtime database roles. The runtime role is not a superuser, table owner bypassing policies, or `BYPASSRLS` role.
- Apply PostgreSQL RLS where tenant data is stored; use `FORCE ROW LEVEL SECURITY` as appropriate. Tenant context is set locally inside a transaction and never persists on a pooled connection.
- RLS supplements application resource/field/action checks. Verify policies using the actual runtime role, including background workers.
- Schema migrations are reviewed and run once per deployment. Use expand/contract changes for rolling releases; document recovery for destructive changes.

See [PostgreSQL row security](https://www.postgresql.org/docs/current/ddl-rowsecurity.html). Graph namespace strings and SQL schema names alone do not establish a security boundary.

### 8.3 Input and output lineage

Every run snapshots the exact eligible versions of its inputs. Store prompt/workflow/schema/model versions, policy version, source dependencies, selected calendar occurrences, normalized user overrides, and the initiating authority. Results reference that snapshot.

An evidence or offer update never silently changes a historical draft. Changes/revocations flag dependent unpublished output as stale. Current action eligibility is rechecked at review/export/publication; a team may regenerate or reapprove under the domain's policy.

Generated output is a candidate derived from evidence. A reference check proves that an ID is supplied and authorized; it does not prove the model's sentence accurately represents the source. Factual review remains a separate gate.

### 8.4 Data onboarding and ingestion

| Step | Required behavior | Owner |
|---|---|---|
| Inventory | Identify source system, data owner, sensitivity, permitted use, expected volume, update/deletion rules | Client/domain owner + integrations |
| Connect | Resolve authorized connection/scopes; encrypt credentials; store source identity | Integrations |
| Extract | Bound file size/type or page batches; use incremental cursors, retries, and deduplication | Integration worker |
| Validate/map | Schema checks, business mapping, quarantined invalid rows, provenance | Domain adapter |
| Store/version | Persist canonical or source-reference records; binaries in private S3; hash revisions | Database/storage |
| Review | Establish validity and allowed internal/public use; unsupported records remain ineligible | Knowledge/domain owner |
| Index | Build keyword/semantic derived indexes only from permitted reviewed data | Knowledge worker |
| Refresh/delete | Advance cursors safely; propagate expiry, revocation, tombstones, deletion, and stale dependency flags | Knowledge/integrations |

Support manual JSON/CSV imports and approved documents before building many vendor connectors. Do not ingest an organization's entire drive automatically. Start with one needed process, one mapped source, and scoped access. Large imports are paginated background jobs with progress, partial errors, and resumable checkpoints.

Document processing treats uploaded files and fetched pages as untrusted data. Apply type/size/time limits, parser isolation where necessary, malicious-content handling, network/URL checks, and provenance. Tenant and document permissions constrain retrieval before returning content to a model. Derived chunks/embeddings follow source deletion and retention rules.

### 8.5 Memory model

| Memory/data kind | Storage and authority | Rules |
|---|---|---|
| Canonical business facts | Versioned domain/evidence records in PostgreSQL | Reviewed applicability and permission; authoritative input |
| Execution state | Run steps and LangGraph checkpoints | Thread/run-scoped; recoverable; not a canonical brand/customer profile |
| Conversations | Optional tenant/user-scoped conversation records | Retention and access policy; never automatic evidence approval |
| Long-term agent memory | Approved records or explicitly governed LangGraph store adapter | Provenance, owner, expiry, correction/deletion; no silent global sharing |
| Semantic index | Optional `pgvector`/derived search index | Rebuildable, tenant-scoped, permission-filtered |
| Factory memory | Sanitized engineering decisions and failure patterns | Separate from production/client information |

LangGraph distinguishes thread-scoped checkpointers from longer-term stores. Blossom must govern both; using either does not grant access or turn model-written memories into approved enterprise facts. See [LangGraph persistence](https://docs.langchain.com/oss/javascript/langgraph/persistence).

## 9. Execution, queues, checkpoints, approvals, and recovery

### 9.1 Asynchronous command path

```mermaid
sequenceDiagram
    participant Web as Web or API consumer
    participant API as Blossom API
    participant DB as PostgreSQL
    participant Delivery as Outbox and BullMQ
    participant Worker as Authorized worker
    Web->>API: Known task and idempotency key
    API->>DB: Validate scope; commit run, snapshot, outbox
    API-->>Web: 202 with runId and status URL
    Delivery->>DB: Claim committed outbox entry
    Delivery->>Worker: Deliver stable job identity
    Worker->>DB: Recheck authority; acquire fenced run lease
    Worker->>Worker: Execute bounded steps and validate outputs
    Worker->>DB: Persist steps, artifacts, usage, state
    Web->>API: Poll authorized run and result
    API-->>Web: Safe progress, findings, result references
```

The outbox is written in the same transaction as the business command. Delivery can repeat. Consumers use a persisted inbox/effect identity, stable artifact identities, and completion guards. A PostgreSQL job registry supports reconciling queue loss; Redis is not the only record that work exists.

Initially return progress through polling. Optional server-sent events can be added when useful, with authorized reconnect and persisted event cursors. Neither polling nor streaming owns the run.

### 9.2 State ownership

| Record | Owns | Does not establish |
|---|---|---|
| Demand | Business request, accountable owner, acceptance/outcome | A workflow is technically finished |
| Run | Authorized execution identity, versions, progress, finalization | External effect certainty without a receipt |
| BullMQ job | Delivery attempt and worker scheduling | Business approval or canonical result |
| Checkpoint | Graph progress and resume cursor | Tenant authority or current release eligibility |
| Content/package revision | Immutable reviewable output | Approval of later edits |
| Approval decision | Exact subject/revision/policy/reviewer decision | Legal signature, payment authority, or another domain's approval |
| External receipt | Evidence of a provider-side outcome | Exactly-once execution across every failure |

Proposed run states: `queued`, `running`, `waiting_for_input`, `succeeded`, `failed`, `cancel_requested`, `cancelled`. Store `waitReason`, such as `human_review` or `missing_input`, separately. A business output may remain `draft` even when the generation run is `succeeded`.

Only authorized application services change states. Expose explicit cancel/retry/resume commands; never accept arbitrary client-supplied run state.

### 9.3 Queue classes and retry ownership

| Queue | Work | Retry owner |
|---|---|---|
| `ai-generation` | Initial text generation and graph execution slices | Worker/runtime; queue retries recover infrastructure failures |
| `research` | Approved current-source research | Research adapter/workflow |
| `ingestion` | Files, source mapping, incremental data sync | Ingestion service with persisted cursor/batch identity |
| `rendering` | Deterministic post/carousel/export artifacts | Rendering service with exact specification/version |
| `publication` | Approved scheduled external publication | Publishing service; reconcile uncertain effects before retry |
| `notifications` | Approved user/domain notifications | Notification adapter with deduplicated delivery |
| `integration-sync` | External receipts, metrics, connector refresh | Connector-specific sync/reconciliation |

Use bounded retry counts, backoff/jitter, deadlines, classified errors, and maximum repair attempts. Avoid multiplying a queue retry budget by an unlimited graph/model retry loop. Invalid inputs and denied policy are terminal/action-required, not transient provider failures. After attempts are exhausted, retain visible failed-job evidence and provide an authorized recovery command.

### 9.4 Recovery requirements

| Failure/edge case | Required behavior |
|---|---|
| Same idempotency key submitted twice | Scope by tenant + operation + key; identical payload returns the original result; changed payload returns `409` |
| Dispatcher crashes after enqueue | Redelivery is safe; stable job identity and persisted effect guards prevent duplicate domain artifacts |
| Worker crash or stale lease | Recover expired lease; increment fencing token; stale workers cannot finalize domain effects |
| Provider `429`/outage | Backoff within deadline/budget; expose failure/queue delay; no endless retries |
| Cancellation during a model call | Stop future steps/effects; already-started calls may finish and cost money; discard or label late results according to policy |
| Queue loss/outage | Rebuild eligible pending delivery from PostgreSQL; expose backlog age and reconciliation status |
| Human review wait | Persist task and run wait state; release the worker; resume only with validated decision/input |
| Changed permissions/connection | Recheck before delayed reads/writes; revoked authority cannot continue consequential work |
| Changed evidence or offer | Keep original snapshot; flag output as stale; enforce current eligibility policy |
| Interrupted graph resumes | Keep the same owned thread and compatible workflow/state versions; use explicit migration/restart when incompatible |
| Graph result saved but run not finalized | Idempotent result materialization/finalizer reconciles artifact references and run state |
| External timeout after sending | Record `needs_reconciliation`; inspect status/receipt before another attempt |
| Large annual/bulk job partially succeeds | Preserve completed months; retry selected failed work within the remaining budget |

Graph checkpoint transactions and business-record transactions may be separate. Do not assume they commit atomically. Each materialization step must be idempotent by run/step/artifact identity; a reconciliation task repairs a checkpoint/result/run-state mismatch.

Provider calls cannot generally be fenced by a database lease after they have left the process. Persist a tool-invocation identity before consequential effects and reconcile ambiguous outcomes. The design offers auditable at-least-once delivery and deduplicated effects where supported, not a universal exactly-once guarantee. See [BullMQ idempotent jobs](https://docs.bullmq.io/patterns/idempotent-jobs).

### 9.5 LangGraph pause/resume boundary

From M5, a runtime adapter compiles an approved graph with durable PostgreSQL persistence and an owned thread binding. All checkpoint operations require trusted tenant/run context. The persistence implementation must enforce tenant isolation across reads, writes, history, listing, and resume; a raw generic saver exposed through an API is unacceptable.

P5-01 selects and validates the compatible implementation: a tenant-aware saver with tenant ownership and policies, or genuinely isolated tenant storage. The official checkpointer can be used behind a suitable adapter only after these properties are demonstrated. Its tables/schema/connection behavior must be inspected; setting a tenant string in configuration is insufficient.

On resume, authorize the actor, read the stored approval/input, verify its subject/revision and current policy, then create a new idempotent execution-slice job. Do not accept a caller's unchecked `approved: true`. LangGraph interrupt resume restarts the interrupted node; operations before the interrupt can run again. Place consequential effects in separate guarded steps. See [LangGraph interrupts](https://docs.langchain.com/oss/javascript/langgraph/interrupts).

### 9.6 Approval and revision policy

Registered approval subjects initially include offer version, evidence version, claim revision, content revision, and plan revision. Publication packages are added in M7. Each subject handler defines eligibility, permissions, validation, separation of duties, expiry, and the exact hash/version to decide on.

An edit creates a new revision with no inherited approval. Historical approved revisions remain readable, but the domain decides whether they remain eligible after new versions or source changes. Unresolved factual/policy findings block release approval. Model-assisted review can create findings; only the authorized service records a decision under the approval policy.

Publication approval binds the exact text revision, caption, ordered media revisions, target account, and material settings in one immutable package. A changed material field requires a new package and applicable approval. Schedules point to a package version, never to `latest` content.

### 9.7 Events

Events crossing modules use registered schemas, a transactional outbox, and consumer inbox deduplication. Example:

```json
{
  "eventId": "evt_example",
  "type": "marketing.content.approved",
  "schemaVersion": 1,
  "tenantId": "workspace_example",
  "aggregateId": "content_example",
  "aggregateVersion": 3,
  "occurredAt": "2026-10-05T16:00:00Z",
  "correlationId": "corr_example",
  "causationId": "approval_example",
  "payload": {
    "revisionId": "revision_example_3",
    "approvalId": "approval_example"
  }
}
```

Payloads prefer authorized references over copied private content. Event consumers still check current eligibility before actions. Initial event families: `demand.*`, `run.*`, `approval.*`, `catalog.offer.*`, `knowledge.evidence.*`, `marketing.content.*`, `integration.connection.*`, and `publication.*`.

## 10. Production agent, workflow, and tool contracts

### 10.1 Agent definition and tenant deployment

Each production definition includes:

| Field | Requirement |
|---|---|
| Identity | Stable `agentKey`, immutable definition version/hash, domain owner |
| Task contracts | Allowlisted task types and input/output schema versions |
| Runtime | Supported runtime identifier and registered workflow versions |
| Instructions | Versioned system/template prompts with input boundaries |
| Tools | Capability allowlist and required scopes; no arbitrary tool discovery/execution |
| Model policy | Approved provider/model choices, fallback conditions, context/token ceilings |
| Recovery policy | Deadlines, retry/repair ceilings, pause/resume/cancellation behavior |
| Approval policy | Required human tasks and consequential-action boundaries |
| Data policy | Allowed sources, classifications, retrieval, memory, retention |
| Evaluation | Fixture/rubric versions and minimum release evidence |
| Operations | Safe progress/result schema, usage attribution, audit, rollback procedure |

A tenant deployment selects an approved definition version, enabled task types, scoped service identity, permissible connections, data scope, and budget. It can narrow authority; it cannot expand a definition beyond reviewed platform policy. Tenant customization uses validated configuration, not uploaded executable JavaScript or unrestricted system prompts.

### 10.2 Internal runtime interface

Illustrative TypeScript contract to implement in `packages/agent-runtime/src/contracts/`. These are Blossom interfaces, not claims about an existing framework API:

```typescript
type RunOutcome =
  | { kind: "completed"; artifactIds: string[] }
  | { kind: "paused"; waitReason: "human_review" | "missing_input"; taskId: string }
  | { kind: "failed"; code: string; retryable: boolean };

interface AuthorizedRunScope {
  tenantId: string;
  runId: string;
  actorId: string;
  agentDeploymentId: string;
  correlationId: string;
}

interface RunContext extends AuthorizedRunScope {
  policyVersion: string;
  leaseToken: number;
  deadlineAt: string;
  budgetReservationId: string;
}

interface RuntimeExecution {
  workflowKey: string;
  workflowVersion: string;
  inputSnapshotId: string;
}

type AuthorizedResume = { expectedStateVersion: number } & (
  | { kind: "decision"; decisionId: string }
  | { kind: "input"; inputRevisionId: string }
);

interface BlossomAgentRuntime {
  execute(ctx: RunContext, execution: RuntimeExecution): Promise<RunOutcome>;
  resume(ctx: RunContext, input: AuthorizedResume): Promise<RunOutcome>;
  requestCancel(scope: AuthorizedRunScope): Promise<void>;
  inspect(scope: AuthorizedRunScope): Promise<{ stateVersion: number; safeState: unknown }>;
}
```

Build `RunContext` from trusted server records, never directly from a model or browser body. Public APIs validate requests into these contracts. The run service owns admission, leases, state, budgets, and finalization; the runtime executes one admitted slice. In-process function execution implements the first adapter. LangGraph implements the advanced adapter. OpenClaw remains a scoped ingress/operator integration initially; a second runtime implementation requires an ADR and the same conformance checks.

### 10.3 Tool invocation contract

Tools are registered capabilities with input/output schemas, owner, risk classification (`read`, `internal_write`, `external_effect`), required scopes, deadline, idempotency/effect policy, allowed connection types, and approval-subject rule if applicable.

The capability gateway performs:

1. Validate tool name and arguments against the deployed definition.
2. Resolve trusted tenant/actor/agent identity and owned resource/connection references.
3. Enforce current grants, data classification, quota, and domain policy.
4. For effects, verify exact eligible approval and persist invocation identity.
5. Invoke a domain service or connector with bounded time and retry rules.
6. Validate result; persist safe receipt/usage/audit; return only permitted data.

MCP tools and LangChain tools use this gateway. Tools never trust model-provided `tenantId`, `approved`, price, role, or credential fields. Read-only tools can still disclose sensitive data, so authorization applies to reads too.

Initial Marketing capabilities are scoped profile lookup, verified calendar lookup, eligible offer/claim lookup, context construction, candidate generation, and revision creation. The writer has no raw Instagram token, arbitrary SQL, shell, or unrestricted HTTP-fetch tool.

### 10.4 Business-agent collaboration

Prefer explicit workflow-defined collaboration. A parent run can request a specialist task through a registered public capability, with child run identity, bounded inputs, remaining budget, timeout, authorized source references, output schema, and cancellation behavior.

For example, Marketing may request an approved offer snapshot from Catalog or a permitted budget-availability response from Finance. It does not receive the company's full ledger. Contracts may return a reviewed permission/obligation reference without exposing unrelated documents. Record parent/child correlation and prevent recursive dispatch/cycles beyond defined depth limits.

A broad supervisor with every department's credentials is not the default architecture. Introduce specialist agents only when evaluation demonstrates improved quality, throughput, or maintainability compared with a deterministic service call.

## 11. Frontend experience and route catalog

### 11.1 Experience requirements

The application has an authorized workspace selector, a domain dashboard, structured request/input forms, work/results views, review inbox, and safe integration/usage settings. Users should understand what is missing, what is running, what needs review, and which result is approved.

Marketing shows a brand brief form, local calendar, Value claims/evidence, annual planner, selected-month draft editor, revision comparison, and a short “Why this narrative?” panel. This panel contains business rationale, source references, assumptions, and findings; it does not expose private model chain-of-thought.

Required UI states: loading, empty, missing input, validation failure, queued/running, waiting for review, stale input, denied action, version conflict, recovered/failed run, cancelled work. Support keyboard interaction, readable contrast, responsive layouts, and locale/timezone display. Keep future unimplemented pages hidden or clearly unavailable.

### 11.2 Planned routes

Next.js dynamic segments are shown with brackets. Protected routes use an authorized active workspace; UI selection does not grant access.

| Route | Screen | Phase |
|---|---|---|
| `/` | Product entry or authenticated redirect | M2 |
| `/sign-in` | Begin OIDC sign-in | P1 |
| `/app` | Workspace dashboard and enabled domains | M3 |
| `/app/workspaces` | Select permitted workspace | P1/M3 |
| `/app/demands` | Request list and supported task intake | P3 |
| `/app/demands/[demandId]` | Inputs, execution, human tasks, accepted outcome | P3 |
| `/app/agents` | Enabled agent capabilities and safe deployment metadata | P5 |
| `/app/workflows` | Approved process catalog and schedules | P5 |
| `/app/knowledge` | Authorized sources, evidence, documents, ingestion status | P4/M6 |
| `/app/runs` | Scoped execution history | P2/M3 |
| `/app/runs/[runId]` | Progress, steps, usage, findings, recovery | P2/M3/M5 |
| `/app/review` | Shared assigned human-task inbox | P2/M4 |
| `/app/marketing/brands` | Brand list/onboarding | M3 |
| `/app/marketing/brands/[brandId]` | Brand overview | M3 |
| `/app/marketing/brands/[brandId]/profile` | Versioned brief and voice | M3 |
| `/app/marketing/brands/[brandId]/audiences` | Needs, objections, buying stage | M3 |
| `/app/marketing/brands/[brandId]/offers` | Approved offer assignments | M3 |
| `/app/marketing/brands/[brandId]/time` | Calendar, seasons, preparation windows | M3 |
| `/app/marketing/brands/[brandId]/value` | Claims, evidence, visible rubric, gaps | M3 |
| `/app/marketing/brands/[brandId]/plans` | Annual-plan list and generation | M4 |
| `/app/marketing/brands/[brandId]/plans/[planId]` | Twelve-month outline and monthly briefs | M4 |
| `/app/marketing/brands/[brandId]/content` | Draft filters, status, freshness | M3 |
| `/app/marketing/brands/[brandId]/content/[contentId]` | Editor, revision comparison, review, export | M3/M4 |
| `/app/marketing/brands/[brandId]/review` | Brand-scoped review tasks | M4 |
| `/app/marketing/brands/[brandId]/research` | Dated signals and review | M6 |
| `/app/marketing/brands/[brandId]/assets` | Creative specs and rendered previews | M7 |
| `/app/marketing/brands/[brandId]/publishing` | Accounts, packages, schedule, pause/reconciliation | M8 |
| `/app/marketing/brands/[brandId]/analytics` | Metrics, windows, missing data, comparisons | M9 |
| `/app/settings` | Workspace policy/settings | P1/M3 |
| `/app/settings/members` | Memberships, roles, assignments, delegation | P1/P4 |
| `/app/settings/usage` | Limits and actual usage | P2/P4 |
| `/app/settings/audit` | Authorized business audit | P2/P4 |
| `/app/integrations` | Connection authorization/status/disconnect | P4/M8 |
| `/app/settings/api-keys` | Scoped service-account credentials | P4 |
| `/app/settings/webhooks` | Client subscriptions/delivery status | P4 |
| `/app/contracts` | Later domain workspace | P6 |
| `/app/finance` | Later domain workspace | P6 |
| `/app/operations` | Later domain workspace | P6 |

The web HTTP handler `/api/blossom/[...path]` forwards allowlisted methods/paths to the fixed NestJS API. Example: `/api/blossom/marketing/brands` → `/v1/marketing/brands`. Authentication callbacks use the same origin. Business generation never runs in this handler. External API consumers call NestJS directly with scoped credentials.

## 12. Shared backend API

### 12.1 API conventions

- Base business prefix: `/v1`. JSON is camelCase; SQL uses snake_case.
- `X-Workspace-Id` selects a workspace which the API resolves against membership/delegation. A tenant-bound API key cannot select a different tenant.
- Validate request bodies/query selectors server-side. Resolve every nested ID under the authorized tenant/resource scope.
- Return `404` for an out-of-scope resource whose existence should remain private, `403` for a visible but denied action, `401` for missing/expired authentication.
- Return `201` for synchronous creation, `202` for admitted asynchronous work, and `204` for documented archive/revoke/delete commands.
- Require `Idempotency-Key` on asynchronous commands and consequential effects. Require `If-Match`/expected version for mutable edits and new revisions derived from a current version.
- Lists use `{ "items": [], "nextCursor": null }`, default `limit=20`, maximum `100`, stable order and opaque cursor. Large exports/imports are background jobs.
- Errors use `{ "code": "STALE_VERSION", "message": "...", "requestId": "...", "details": [] }`. Use `400` for invalid input, `409` for version/idempotency/state conflict, `429` for quotas/rate limits; document dependency failures.
- `DELETE` normally archives a record or revokes access. Retention-driven permanent erasure is a separate audited process.
- `202` includes `runId`, `status`, `statusUrl`, and safe result references when available. A generation admission creates the run and snapshot before returning.
- Public callbacks/webhooks validate state/signatures and resolve the owning connection; a body/header tenant value is not trusted.
- The generated OpenAPI file is the executable contract for implemented routes. Planned routes stay labeled in requirements until they ship.

Access labels below: **user** = authenticated and scoped; **admin** = appropriate workspace grant; **ops** = platform operations; named editor/approver/operator = domain grant; **service** = scoped service principal; **public** = narrow protocol entry only.

### 12.2 Health and identity

| Method | Route | Responsibility/access | Phase |
|---|---|---|---|
| GET | `/health/live` | Process alive, no sensitive diagnostics; public | P1 |
| GET | `/health/ready` | Dependencies ready; internal/ops | P1/P2 |
| GET | `/v1/auth/login` | OIDC flow with approved redirect; public | P1 |
| GET | `/v1/auth/callback` | Validate code/state and issue application session; protocol callback | P1 |
| GET | `/v1/auth/session` | Safe current-session summary | P1 |
| GET | `/v1/auth/csrf` | Session-bound CSRF token | P1 |
| POST | `/v1/auth/logout` | Revoke session and clear cookie; user | P1 |
| GET | `/v1/me` | Current identity and authorized workspaces; user | P1 |
| GET | `/v1/openapi.json` | Implemented API specification; dev/authorized consumer | P1 |
| GET | `/docs` | Swagger UI; dev/staging or authorized access | P1 |

Refresh is server-side session/OIDC logic, not a browser endpoint exposing tokens. Account invitations/provisioning initially use the identity provider's reviewed administrative process.

### 12.3 Workspaces and customer registry

| Method | Route | Responsibility/access | Phase |
|---|---|---|---|
| GET | `/v1/workspaces` | List authorized tenants; user | P1 |
| POST | `/v1/workspaces` | Provision isolated workspace; ops | P1 |
| GET | `/v1/workspaces/:workspaceId` | Read permitted workspace | P1 |
| PATCH | `/v1/workspaces/:workspaceId` | Version-guarded settings; admin | P1 |
| GET | `/v1/workspaces/:workspaceId/members` | Permitted members; admin | P1 |
| POST | `/v1/workspaces/:workspaceId/members` | Grant membership to resolved existing identity; admin | P1 |
| PATCH | `/v1/workspaces/:workspaceId/members/:memberId` | Change permitted roles/resource assignments; admin | P1 |
| DELETE | `/v1/workspaces/:workspaceId/members/:memberId` | Revoke access and delayed-action eligibility; admin | P1 |
| POST | `/v1/workspaces/:workspaceId/delegations` | Scoped/expiring staff or support grant; authorized policy | P4 |
| DELETE | `/v1/workspaces/:workspaceId/delegations/:delegationId` | Revoke delegation | P4 |
| GET | `/v1/workspaces/:workspaceId/org-units` | List authorized departments/teams | P6 |
| POST | `/v1/workspaces/:workspaceId/org-units` | Create unit; admin | P6 |
| PATCH | `/v1/workspaces/:workspaceId/org-units/:unitId` | Version-guarded unit update; admin | P6 |
| DELETE | `/v1/workspaces/:workspaceId/org-units/:unitId` | Archive unit; admin | P6 |
| GET | `/v1/customer-accounts` | Permitted commercial references | P1 |
| POST | `/v1/customer-accounts` | Create approved company/account reference; admin | P1 |
| GET | `/v1/customer-accounts/:accountId` | Read permitted reference | P1 |
| PATCH | `/v1/customer-accounts/:accountId` | Update commercial metadata; admin | P1 |

### 12.4 Demands, agent registry, and process catalog

| Method | Route | Responsibility/access | Phase |
|---|---|---|---|
| GET | `/v1/demands` | Scoped request list/filter | P3 |
| POST | `/v1/demands` | Submit supported structured request; user/service | P3 |
| GET | `/v1/demands/:demandId` | Inputs, owner, state, outcome and run references | P3 |
| PATCH | `/v1/demands/:demandId` | Allowed fields/assignment with expected version | P3 |
| POST | `/v1/demands/:demandId/submissions` | Supply missing information as a new input revision | P3 |
| POST | `/v1/demands/:demandId/qualification` | Validate supported task and qualify; domain operator | P3 |
| POST | `/v1/demands/:demandId/executions` | Admit qualified task; operator/service; `202` | P3 |
| POST | `/v1/demands/:demandId/resolutions` | Accept/reject business outcome; authorized owner | P3 |
| POST | `/v1/demands/:demandId/cancel` | Cancel demand and request linked-run cancellation | P3 |
| GET | `/v1/agents` | Enabled capabilities/supported task types | P2 |
| GET | `/v1/agents/:agentId` | Safe definition metadata and required inputs | P2 |
| GET | `/v1/agents/:agentId/versions` | Approved version metadata | P5 |
| GET | `/v1/agent-deployments` | Tenant-enabled definitions/configuration | P5 |
| POST | `/v1/agent-deployments` | Enable reviewed definition with narrowed grants; admin | P5 |
| PATCH | `/v1/agent-deployments/:deploymentId` | Version-guarded allowed configuration; admin | P5 |
| DELETE | `/v1/agent-deployments/:deploymentId` | Disable deployment/future tasks; admin | P5 |
| GET | `/v1/workflows` | Approved tenant process catalog | P5 |
| GET | `/v1/workflows/:workflowId` | Task contract, versions, review requirements | P5 |

Registry code is released through CI; no public endpoint uploads executable agents or graphs. Marketing's direct generation endpoint and demand executions call the same admission use case, so they do not create competing execution implementations.

### 12.5 Offers, evidence, and dates

| Method | Route | Responsibility/access | Phase |
|---|---|---|---|
| GET | `/v1/offers` | Scoped catalog list | P1/M3 |
| POST | `/v1/offers` | Create identity; catalog editor | P1/M3 |
| GET | `/v1/offers/:offerId` | Identity/current approved version | P1/M3 |
| POST | `/v1/offers/:offerId/versions` | New immutable benefit/price/availability version; editor | P1/M3 |
| GET | `/v1/offers/:offerId/versions` | Version history | P1/M3 |
| GET | `/v1/offers/:offerId/versions/:versionId` | Exact version | P1/M3 |
| DELETE | `/v1/offers/:offerId` | Archive offer; editor | P1/M3 |
| GET | `/v1/evidence` | Scoped evidence/source list | P1/M3 |
| POST | `/v1/evidence` | Create source/evidence record; evidence editor | P1/M3 |
| GET | `/v1/evidence/:evidenceId` | Permitted source fields | P1/M3 |
| POST | `/v1/evidence/:evidenceId/versions` | Immutable applicability/validity/public-use version | P1/M3 |
| GET | `/v1/evidence/:evidenceId/versions` | Evidence history | P1/M3 |
| GET | `/v1/evidence/:evidenceId/versions/:versionId` | Exact permitted version | P1/M3 |
| POST | `/v1/evidence/:evidenceId/revocations` | Withdraw use with reason; evidence approver | P1/M3 |
| GET | `/v1/calendar/events` | Applicable event definitions | P1/M3 |
| POST | `/v1/calendar/events` | Workspace/brand event; calendar editor | P1/M3 |
| GET | `/v1/calendar/events/:eventId` | Source, verification, recurrence | P1/M3 |
| PATCH | `/v1/calendar/events/:eventId` | Version-guarded update preserving prior references | P1/M3 |
| DELETE | `/v1/calendar/events/:eventId` | Archive custom event | P1/M3 |
| POST | `/v1/calendar/imports` | Validate/import reviewed occurrences; calendar editor | M3 |
| GET | `/v1/calendar/occurrences` | Resolve market/year/date range | M3 |

Offer/evidence approval uses the shared subject registry. Mutable metadata cannot rewrite a saved input snapshot.

### 12.6 Documents, sources, and ingestion

| Method | Route | Responsibility/access | Phase |
|---|---|---|---|
| GET | `/v1/documents` | Scoped metadata list | M6/P4 |
| POST | `/v1/documents/uploads` | Authorize size/type and issue pending signed upload | M6/P4 |
| POST | `/v1/documents/uploads/:uploadId/complete` | Verify stored bytes/hash and queue ingestion | M6/P4 |
| GET | `/v1/documents/:documentId` | Authorized metadata/processing state | M6/P4 |
| GET | `/v1/documents/:documentId/download` | Authorize short-lived download | M6/P4 |
| DELETE | `/v1/documents/:documentId` | Archive and initiate retention/dependency updates | M6/P4 |
| GET | `/v1/knowledge/sources` | Permitted source inventory | P4 |
| POST | `/v1/knowledge/sources` | Register approved source and use policy; admin | P4 |
| PATCH | `/v1/knowledge/sources/:sourceId` | Version-guarded source policy/configuration | P4 |
| POST | `/v1/knowledge/sources/:sourceId/ingestion-runs` | Queue supported bounded import/sync | P4 |
| GET | `/v1/knowledge/ingestion-runs/:runId` | Scoped ingestion progress/errors | P4 |
| POST | `/v1/knowledge/search` | Bounded authorized query with source/field filters | P4/M6 |

Source registration accepts supported configurations, not arbitrary server code, SQL, filesystem paths, or unrestricted URL credentials. Document upload completion verifies ownership and content; a caller cannot register another tenant's object key.

### 12.7 Runs, human tasks, approvals, and schedules

| Method | Route | Responsibility/access | Phase |
|---|---|---|---|
| GET | `/v1/runs` | Scoped run list | P2 |
| GET | `/v1/runs/:runId` | State, safe error, usage, result references | P2 |
| GET | `/v1/runs/:runId/steps` | Safe step timeline/findings | P2/M5 |
| POST | `/v1/runs/:runId/cancel` | Cooperative cancellation; owner/operator | P2 |
| POST | `/v1/runs/:runId/retry` | Eligible failed work under current authority/budget | P2/M5 |
| POST | `/v1/runs/:runId/resume` | Authorized stored decision/input; no unchecked approval | M5 |
| GET | `/v1/review-tasks` | Assigned/authorized human tasks | P2/M4 |
| GET | `/v1/review-tasks/:taskId` | Exact subject, findings, assignment | P2/M4 |
| PATCH | `/v1/review-tasks/:taskId` | Authorized assignment/due-date update | P2/M4 |
| POST | `/v1/approvals` | Request registered subject + exact revision review | P2/M3 |
| GET | `/v1/approvals` | Scoped approval list | P2/M4 |
| GET | `/v1/approvals/:approvalId` | Subject, policy, outcome, audit | P2/M4 |
| POST | `/v1/approvals/:approvalId/decisions` | Approve/reject/request changes under domain policy | P2/M3 |
| POST | `/v1/approvals/:approvalId/revoke` | Withdraw eligibility with reason | P2/M4 |
| GET | `/v1/schedules` | Authorized workflow schedules | P5/M5 |
| POST | `/v1/schedules` | Known workflow + timezone + policy; admin/operator | P5/M5 |
| PATCH | `/v1/schedules/:scheduleId` | Version-guarded change/pause | P5/M5 |
| DELETE | `/v1/schedules/:scheduleId` | Disable future occurrences | P5/M5 |

There is no unrestricted execute-prompt/tool endpoint. Retry/resume always resolve a registered run/definition and compatible version.

### 12.8 Connections, usage, audit, and external connectivity

| Method | Route | Responsibility/access | Phase |
|---|---|---|---|
| GET | `/v1/integrations` | Safe connection metadata | P4/M6 |
| POST | `/v1/integrations/:provider/authorization` | Start allowlisted OAuth flow; connection admin | P4/M8 |
| GET | `/v1/integrations/:provider/callback` | Validate state and resolve owning connection | P4/M8 |
| GET | `/v1/integrations/:integrationId/status` | Scopes/expiry/last sync/safe diagnostics | P4/M8 |
| DELETE | `/v1/integrations/:integrationId` | Disconnect/revoke and prevent future actions | P4/M8 |
| GET | `/v1/webhooks/instagram` | Provider verification challenge | M8 |
| POST | `/v1/webhooks/instagram` | Verify signature; deduplicate/enqueue minimal delivery | M8 |
| GET | `/v1/usage/summary` | Tenant/run/token/tool/storage usage for window | P2 |
| GET | `/v1/usage/limits` | Active entitlements | P2 |
| PATCH | `/v1/workspaces/:workspaceId/limits` | Approved quota/budget changes; ops | P4 |
| GET | `/v1/audit-events` | Scoped business history; audit reader | P2/P4 |
| POST | `/v1/service-accounts` | Create narrow tenant automation identity; admin | P4 |
| GET | `/v1/service-accounts` | List scoped identities | P4 |
| POST | `/v1/service-accounts/:serviceAccountId/api-keys` | Expiring credential shown once; admin | P4 |
| DELETE | `/v1/service-accounts/:serviceAccountId/api-keys/:keyId` | Revoke credential | P4 |
| POST | `/v1/webhook-subscriptions` | Approved event destination/scopes; admin | P4 |
| GET | `/v1/webhook-subscriptions` | List permitted subscriptions | P4 |
| PATCH | `/v1/webhook-subscriptions/:subscriptionId` | Update/pause permitted configuration | P4 |
| DELETE | `/v1/webhook-subscriptions/:subscriptionId` | Disable future delivery | P4 |
| GET | `/v1/webhook-subscriptions/:subscriptionId/deliveries` | Safe status/attempt history | P4 |

Outgoing client webhooks require network/URL validation, signed payloads, secret rotation, bounded retries, event schema versions, and replay/deduplication guidance. Public provider callbacks never trust a user-provided tenant ID.

Future `/v1/contracts/*`, `/v1/finance/*`, and `/v1/operations/*` are reserved boundaries. P6 must define their actual routes from owned process requirements. They are not automatically installed Marketing tools.

## 13. Marketing: Time, Value, and the first application

### 13.1 Scope and input contract

Marketing is the first production domain. It combines **Time**—why this matters during a period—with **Value**—why a particular audience should choose an offer. Initially these are typed TypeScript capabilities inside a controlled workflow, not separate autonomous agents.

The first business inputs remain brand/client information and a verified calendar. Audience, offers, claims, and supporting evidence are structured parts of the brand brief, not an invitation to crawl the internet or the client's systems during M0.

| Input | Required fields/meaning |
|---|---|
| Brand profile | Name, sector, description, approved positioning, market, IANA timezone, locale, language, tone, prohibited terms/topics |
| Audience | Stable ID, need, desired outcome, objections, location/applicability, buying stage |
| Offer version | ID/version, benefit/mechanism, eligible CTA, price/currency when used, availability/capacity/validity |
| Claim revision | Exact wording/meaning, applicability, eligible evidence links, review state, expiry |
| Evidence version | Source/reference, observation, owner, method/review date, validity, classification, public-use permission |
| Calendar | Verified local occurrences, source/version, local seasons, preparation/peak/follow-up windows |
| Campaign settings | Objective, year/month, selected audience/offer, draft count, channel intent, approved promotion, excluded dates |

Pilot data: one brand, one audience, one offer, three supported claims, about ten relevant annual events, and one selected month. Synthetic fixtures can prove software behavior; real pilot facts must be verified by the marketing owner.

### 13.2 Time capability

`buildTimeContext(input): TimeContext` is orchestration-independent. It selects applicable verified occurrences, local season context, campaign windows, source references, relevance notes, and missing-information findings.

| Dimension | Meaning | First-release source | Later enhancement |
|---|---|---|---|
| Calendar/season | Market-specific holidays, local periods, client launches | Curated reviewed records | Verified feeds or dedicated campaign calendar |
| Recurring opportunity | Black Friday, gifting periods, known annual industry events | Selected applicable calendar events | Campaign/client history |
| Trend | Observed current attention/behavior change | Only dated approved client evidence, if supplied | Reviewed current research in M6 |
| Innovation | New capability/product/market development | Approved launch information | Reviewed current research in M6 |

Requirements:

- Represent market by country/subdivision/locality where relevant, not language alone.
- Use verified concrete local occurrences for the selected year. Resolve movable dates in deterministic code or reviewed imports; the model is not the authority for dates.
- Configure locally appropriate seasons, including wet/dry periods where relevant. Do not assume northern-hemisphere four-season dates.
- Use local date strings for all-day events. Scheduling later stores both UTC instant and original timezone; reject ambiguous/nonexistent local times unless explicitly resolved.
- Record lead time, peak, follow-up, exclusions, and capacity constraints.
- Treat an event as a possible opportunity, not proof of demand. Record audience relevance separately.
- Return an evergreen opportunity when no applicable occasion exists. Missing market requires correction or explicitly evergreen-only output.
- A static calendar does not predict emerging trends a year in advance.

Shared date parsing, timezone utilities, occurrence verification, and scheduler infrastructure can serve Finance/Contracts. Holiday-to-sales storytelling remains Marketing-owned.

### 13.3 Value capability

`buildValueContext(input): ValueContext` selects the audience need, exact eligible offer version, benefit/mechanism, approved claim/evidence IDs, verified comparisons when available, explicit assumptions/gaps, and a short business rationale.

Value statement structure:

> For **[audience]** with **[need]**, **[offer]** provides **[outcome]** through **[mechanism]**, supported by **[evidence]**, with **[verified relevant difference]**.

Maintain company claims, verified outcomes, customer perceptions, and competitor evidence as distinct records. Missing comparative data means unknown. Price, brand identity, or engagement alone cannot prove superiority or customer value.

| Dimension | Practical measure | Evidence/input | Use |
|---|---|---|---|
| Need importance | Interview frequency or rated importance with sample | Customer research and sales objections | Choose relevant audience need |
| Functional outcome | Time saved, cost avoided, reliability, delivery, durability | Tests or measured customer outcomes | State benefit with units/conditions |
| Comparative difference | Dated comparison with relevant alternatives | Approved specifications/research | Explain verifiable differences |
| Evidence coverage | Supported factual claims / claims checked | Claim/evidence mapping plus review | Identify factual gaps |
| Evidence strength | Applicability, method, recency, credibility rubric | Tests, case studies, credentials | Prefer evidence matching exact offer/audience |
| Perceived value | Preference/reasons for purchase with method/sample | Customer research | Compare intended vs perceived benefit |
| Willingness to pay | Actual paid prices or valid pricing research | Transactions/research | Assess paid demand; do not infer from likes |
| Commercial feasibility | Approved capacity, availability, margin indicator | Catalog/operations/Finance-approved fields | Avoid unavailable/unviable promotions |
| Qualified demand | Qualified inquiries/appointments under defined denominator | CRM or agreed manual tracking | Connect narrative with useful demand |
| Conversion | Purchases / defined eligible leads/visits | Commerce/CRM tracking with attribution window | Evaluate commercial performance |
| Relationship value | Retention, repeat purchase, referral, customer outcome | Customer history | Assess lasting value over an appropriate cycle |

Use measures appropriate to the client's industry. Missing data remains visible. The agent must not invent numeric results, testimonials, discounts, scarcity, credentials, or competitor facts.

Initial editorial rubric uses human-reviewed 0–5 component scores:

`priority = 0.35 × audienceRelevance + 0.25 × timeRelevance + 0.25 × evidenceStrength + 0.15 × feasibility`

Store the rubric version, component notes, and weights. These proposed weights rank editorial opportunities; they are not validated sales forecasts or a universal intrinsic-value formula. A high score cannot override expired/unapproved evidence, unavailable offers, denied access, or prohibited claims.

Internal reasoning permission and public-use permission are independent. Private evidence may inform strategy without authorizing a quotation, testimonial identity, or confidential disclosure. Releasable factual wording must use eligible support and permitted disclosure.

### 13.4 Output contracts

Persist schema-validated structured JSON and render it as readable text. Model outputs contain candidate content fields and references from the allowed input set. The server supplies trusted tenant/record IDs, eligibility findings, usage, provenance, and status.

| Artifact | Required structure | Validation |
|---|---|---|
| Annual outline | Year, strategy summary, exactly twelve unique months, each with audience/offer, objective, theme, Time/Value references, CTA direction | Complete year, supported references, coherent variation, realistic preparation windows |
| Monthly brief | Parent plan revision, period, audience need/objective, selected Time/Value context, angle, CTA, assumptions | Eligible strategy relationship, valid dates/offer/evidence |
| Text draft | Hook, body, CTA, channel intent, period, references, business rationale, assumptions | Schema/length/language, prohibited terms, reference/date checks, factual review |
| Creative specification | Template/version, ordered slides, text bounds, media IDs, caption, accessibility text | Exact reviewed narrative link, safe template, layout/channel validation |
| Publication package | Exact text/media/caption/account/material settings and policy | Current approval, account scope, source/offer validity, duplicate/effect guards |

Persisted draft illustration; all identifiers/content are synthetic:

```json
{
  "schemaVersion": 1,
  "tenantId": "workspace_demo",
  "brandId": "brand_demo",
  "contentId": "content_demo",
  "revisionId": "revision_demo_1",
  "period": { "year": 2027, "month": 1 },
  "audienceId": "audience_demo",
  "offerVersionId": "offer_demo_v1",
  "objective": "consideration",
  "eventOccurrenceIds": [],
  "claimRevisionIds": ["claim_demo_r1"],
  "evidenceVersionIds": ["evidence_demo_v1"],
  "narrativeAngle": "An evergreen audience need",
  "hook": "Make your daily routine more practical.",
  "body": "Explore the approved offer and its documented features.",
  "callToAction": "Read the product details.",
  "businessRationale": "No relevant verified event was selected; the brief uses an evergreen need.",
  "assumptions": [],
  "validation": {
    "referenceChecksPassed": true,
    "unresolvedFindings": [],
    "requiresHumanReview": true
  },
  "provenance": {
    "runId": "run_demo",
    "inputSnapshotId": "snapshot_demo",
    "brandProfileVersionId": "profile_demo_v1",
    "promptVersion": "marketing-monthly-v1",
    "workflowVersion": "marketing-text-v1",
    "modelId": "record_actual_selected_model"
  },
  "status": "draft"
}
```

`validation` and `status` are server/reviewer fields. The model cannot self-certify them. A generation run can complete with reviewable candidate text; unresolved factual findings prevent release approval.

### 13.5 Workflow

```mermaid
flowchart TD
    Input["Approved brand and calendar snapshot"] --> Scope["Validate scope, budget, inputs"]
    Scope --> Time["Build Time context"]
    Scope --> Value["Build Value context"]
    Time --> Generate["Annual outline or monthly drafts"]
    Value --> Generate
    Generate --> Validate["Schema, references, facts, policy findings"]
    Validate -->|"Repair eligible and budget remains"| Repair["Bounded targeted repair"]
    Repair --> Validate
    Validate -->|"Reviewable"| Review["Save exact revision and review task"]
    Validate -->|"Unresolved blocking findings"| Blocked["Blocked output with visible findings"]
    Review -->|"Changes requested"| Edit["New edited or regenerated revision"]
    Edit --> Validate
    Review -->|"Approved and eligible"| Export["Approved text export"]
    Export -.-> Creative["Later: render exact media package"]
    Creative -.-> Publish["Later: approved scheduled publication"]
```

M0–M4 implement an explicit application sequence. M5 ports steps that benefit from checkpointing, branching, and review pause/resume to LangGraph, preserving the same product contracts and evaluations.

The annual output is a strategy outline. Expand detailed text on a rolling monthly basis, initially four to eight drafts for a selected month. Optional full-year detailed batches are introduced only after monthly quality, cost, and recovery are understood. They use bounded child jobs and selective regeneration.

### 13.6 User controls

Editors select authorized year/month, audience, objective, offer, approved claims, event emphasis/evergreen mode, tone/language/length within policy, permitted CTA, and draft count. Strategists review voice/claims, strategy, editorial weights, and exact outputs. Admins configure budgets, enabled agents, source policies, and integrations.

Model choice, privileged system prompts, and tool permissions are controlled, versioned configuration changes subject to evaluation. Do not expose unrestricted prompt editing as a way to change business permissions.

Edits save new revisions; stale/approved status and source references remain visible. Export can include a draft only if clearly labeled and the user has export permission. Approved export selects an exact approved eligible revision, never the latest unreviewed text.

### 13.7 Advanced Marketing

| Milestone | Requirements | Release gate |
|---|---|---|
| M5 durable planning | Owned checkpoints, review resume, dependency/freshness tracking, rolling refresh | Restart/resume preserves work and approval boundaries |
| M6 current research | Allowlisted search/fetch, dated signals, source/market/applicability/expiry, strategist review | Distinguish calendar facts, claims, observations, hypotheses; unavailable/malicious sources handled |
| M7 creative | Approved post/carousel templates, exact text/spec versions, private assets, isolated rendering | No clipping/overflow; designer reviews exact rendered package |
| M8 Instagram | Official API, eligible account, current OAuth/scopes, package approval, scheduler, receipts | Reviewed test post and timeout/reconciliation drill; current platform access confirmed |
| M9 measurement | Defined metrics, windows/denominators, available Instagram/CRM/manual data, provenance | Reports expose missing data and attribution limits; reviewed changes pass regression evaluation |

Static template rendering controls exact typography and text. Optional image generation may supply suitable imagery later; it does not lay out precise captions/slides. An Instagram feed publication needs eligible media; a text-only narrative is the foundation, not a complete feed post.

Publication states: `scheduled`, `preparing`, `publishing`, `published`, `failed`, `cancelled`, `needs_reconciliation`. Store external container/media IDs and receipts. Check current professional-account/API permissions, media constraints, quotas, and external-client review requirements during M8 rather than hard-coding assumptions from this document.

The improvement loop updates reviewed inputs, prompts, and rules. It is not automatic model retraining. Engagement metrics alone cannot establish intrinsic value or causally prove sales improvement. State qualified-lead definitions, attribution windows, denominators, and missing coverage; use controlled experiments where feasible.

## 14. Marketing backend API

All routes inherit section 12 conventions. Prefix `/v1/marketing`. Shared offers, evidence, calendar, documents, approvals, integrations, and runs remain in their shared modules. Phase tags identify implementation scope; future routes are backlog contracts.

### 14.1 Brands, audiences, claims, context

| Method | Route | Responsibility/access | Phase |
|---|---|---|---|
| GET | `/v1/marketing/brands` | Authorized brand list | M3 |
| POST | `/v1/marketing/brands` | Create brand; admin/brand manager | M3 |
| GET | `/v1/marketing/brands/:brandId` | Summary/current profile | M3 |
| PATCH | `/v1/marketing/brands/:brandId` | Version-guarded identity/settings | M3 |
| DELETE | `/v1/marketing/brands/:brandId` | Archive/disable new work | M3 |
| POST | `/v1/marketing/brands/:brandId/profile-versions` | Immutable validated brief; editor | M3 |
| GET | `/v1/marketing/brands/:brandId/profile-versions` | Brief history | M3 |
| GET | `/v1/marketing/brands/:brandId/profile-versions/:versionId` | Exact permitted brief | M3 |
| GET | `/v1/marketing/brands/:brandId/audiences` | Audience list | M3 |
| POST | `/v1/marketing/brands/:brandId/audiences` | Create audience; editor | M3 |
| PATCH | `/v1/marketing/brands/:brandId/audiences/:audienceId` | Version-guarded edit; preserve snapshots | M3 |
| DELETE | `/v1/marketing/brands/:brandId/audiences/:audienceId` | Archive audience | M3 |
| GET | `/v1/marketing/brands/:brandId/offer-assignments` | Eligible shared catalog assignments | M3 |
| POST | `/v1/marketing/brands/:brandId/offer-assignments` | Assign permitted offer; manager | M3 |
| DELETE | `/v1/marketing/brands/:brandId/offer-assignments/:assignmentId` | Remove assignment | M3 |
| GET | `/v1/marketing/brands/:brandId/claims` | Claims/evidence/review state | M3 |
| POST | `/v1/marketing/brands/:brandId/claims` | Propose identity and first revision; editor | M3 |
| GET | `/v1/marketing/brands/:brandId/claims/:claimId` | Detail and revision references | M3 |
| POST | `/v1/marketing/brands/:brandId/claims/:claimId/revisions` | Immutable wording/scope/evidence revision | M3 |
| POST | `/v1/marketing/brands/:brandId/claims/:claimId/revoke` | Withdraw eligible use; strategist | M3 |
| GET | `/v1/marketing/brands/:brandId/time-context` | Verified context for bounded year/month/market | M3 |
| GET | `/v1/marketing/brands/:brandId/value-context` | Eligible offer/audience/claims, scores/gaps | M3 |
| GET | `/v1/marketing/brands/:brandId/policies` | Narrative restrictions/defaults/rubric | M3 |
| PATCH | `/v1/marketing/brands/:brandId/policies` | Versioned permitted policy/rubric edit | M3/M4 |

Context selectors include `year`, `month`, `audienceId`, and `offerVersionId`; the server validates each reference. Claim approvals use the shared approval API and exact revision.

### 14.2 Plans, generation, content, export

| Method | Route | Responsibility/access | Phase |
|---|---|---|---|
| POST | `/v1/marketing/brands/:brandId/generation-runs` | Admit `monthly_drafts` or `annual_outline`; editor; `202` | M3/M4 |
| GET | `/v1/marketing/brands/:brandId/plans` | Annual-plan list | M4 |
| GET | `/v1/marketing/brands/:brandId/plans/:planId` | Plan and twelve monthly briefs | M4 |
| GET | `/v1/marketing/brands/:brandId/plans/:planId/revisions` | Plan history | M4 |
| POST | `/v1/marketing/brands/:brandId/plans/:planId/revisions` | Edited immutable plan revision | M4 |
| GET | `/v1/marketing/brands/:brandId/plans/:planId/months/:month` | Selected brief, scoped to parent plan revision | M4 |
| POST | `/v1/marketing/brands/:brandId/plans/:planId/month-regeneration-runs` | Selective refresh with parent revision | M5 |
| GET | `/v1/marketing/brands/:brandId/content` | Draft list/status/period/freshness | M3 |
| POST | `/v1/marketing/brands/:brandId/content` | Manual candidate draft with provenance; editor | M3 |
| GET | `/v1/marketing/brands/:brandId/content/:contentId` | Current text, version, eligibility | M3 |
| POST | `/v1/marketing/brands/:brandId/content/:contentId/revisions` | Immutable edit with expected version | M3 |
| GET | `/v1/marketing/brands/:brandId/content/:contentId/revisions` | Revision history | M3 |
| GET | `/v1/marketing/brands/:brandId/content/:contentId/revisions/:revisionId` | Exact text/input references/findings | M3 |
| GET | `/v1/marketing/brands/:brandId/content/:contentId/compare` | Authorized selected revision comparison | M4 |
| POST | `/v1/marketing/brands/:brandId/content/:contentId/regeneration-runs` | Full or allowlisted-section regeneration | M4/M5 |
| DELETE | `/v1/marketing/brands/:brandId/content/:contentId` | Archive draft | M3 |
| GET | `/v1/marketing/brands/:brandId/plans/:planId/export` | Exact plan revision as `md`/`json`; export grant | M4 |
| GET | `/v1/marketing/brands/:brandId/content/:contentId/export` | Exact text revision as `md`/`json`; export grant | M4 |

Monthly briefs are initially versioned inside the plan revision. Editing a brief creates a new parent plan revision. Resolve export/compare/month selectors explicitly; require exact revision selection for approved exports.

Generation example after the synthetic records have been persisted:

```http
POST /v1/marketing/brands/brand_demo/generation-runs
X-Workspace-Id: workspace_demo
Idempotency-Key: example-client-request-001
Content-Type: application/json
```

```json
{
  "taskType": "monthly_drafts",
  "year": 2027,
  "month": 1,
  "brandProfileVersionId": "profile_demo_v1",
  "audienceId": "audience_demo",
  "offerVersionId": "offer_demo_v1",
  "objective": "consideration",
  "draftCount": 4,
  "overrides": {
    "tone": "clear and practical",
    "language": "en",
    "useEvergreenIfNoRelevantEvent": true
  }
}
```

```json
{
  "runId": "run_demo",
  "status": "queued",
  "statusUrl": "/v1/runs/run_demo"
}
```

The server resolves eligible claim/calendar versions, normalizes allowed overrides, reserves budget, and creates the full snapshot. Reject unbounded instructions or privileged prompt/tool overrides. Shared run endpoints supply progress/cancellation/recovery.

### 14.3 Research, creative assets, packages

| Method | Route | Responsibility/access | Phase |
|---|---|---|---|
| POST | `/v1/marketing/brands/:brandId/research-runs` | Bounded approved sector/trend/competitor question | M6 |
| GET | `/v1/marketing/brands/:brandId/signals` | Signals by source/topic/review/expiry | M6 |
| POST | `/v1/marketing/brands/:brandId/signals` | Manually sourced dated signal | M6 |
| GET | `/v1/marketing/brands/:brandId/signals/:signalId` | Observation/source/applicability/findings | M6 |
| POST | `/v1/marketing/brands/:brandId/signals/:signalId/reviews` | Accept/reject under source policy; strategist | M6 |
| GET | `/v1/marketing/brands/:brandId/creative-templates` | Approved templates/versions/limits | M7 |
| POST | `/v1/marketing/brands/:brandId/creative-templates` | Register allowlisted template configuration; designer/admin | M7 |
| POST | `/v1/marketing/brands/:brandId/content/:contentId/creative-runs` | Render exact reviewed narrative/specification | M7 |
| GET | `/v1/marketing/brands/:brandId/assets` | Authorized asset/spec list | M7 |
| GET | `/v1/marketing/brands/:brandId/assets/:assetId` | Metadata/revisions/authorized preview | M7 |
| POST | `/v1/marketing/brands/:brandId/assets/:assetId/revisions` | Validated spec or authorized upload reference | M7 |
| POST | `/v1/marketing/brands/:brandId/publication-packages` | Bind exact text/caption/media/account/material settings | M7/M8 |
| GET | `/v1/marketing/brands/:brandId/publication-packages/:packageId` | Exact package, version, approval/eligibility | M7/M8 |

Renderer executable code is reviewed and deployed from the repository. Template registration accepts a permitted identifier/configuration, never arbitrary JavaScript from a user/model. New template/media/caption versions produce new reviewable package identity.

### 14.4 Publication and measurement

| Method | Route | Responsibility/access | Phase |
|---|---|---|---|
| GET | `/v1/marketing/brands/:brandId/publishing-accounts` | Scoped eligible connected accounts | M8 |
| POST | `/v1/marketing/brands/:brandId/publishing-accounts` | Assign authorized external account to brand | M8 |
| DELETE | `/v1/marketing/brands/:brandId/publishing-accounts/:accountId` | Remove assignment and future eligibility | M8 |
| POST | `/v1/marketing/brands/:brandId/publishing-jobs` | Schedule exact approved package/account/time | M8 |
| GET | `/v1/marketing/brands/:brandId/publishing-jobs` | Schedule/status list | M8 |
| GET | `/v1/marketing/brands/:brandId/publishing-jobs/:jobId` | Attempts, receipts, failure/reconciliation | M8 |
| POST | `/v1/marketing/brands/:brandId/publishing-jobs/:jobId/cancel` | Cancel eligible future effect | M8 |
| POST | `/v1/marketing/brands/:brandId/publishing-jobs/:jobId/retry` | Eligible retry after duplicate/outcome checks | M8 |
| POST | `/v1/marketing/brands/:brandId/publishing-jobs/:jobId/reconcile` | Queue provider-status/receipt inspection | M8 |
| GET | `/v1/marketing/brands/:brandId/publishing-policy` | Pause/account/publication policy | M8 |
| PATCH | `/v1/marketing/brands/:brandId/publishing-policy` | Authorized pause/resume | M8 |
| PATCH | `/v1/workspaces/:workspaceId/publishing-policy` | Tenant-wide pause/resume; admin | M8 |
| POST | `/v1/marketing/brands/:brandId/metric-sync-runs` | Collect available platform/CRM metrics | M9 |
| GET | `/v1/marketing/brands/:brandId/metrics` | Definitions, sources, windows, denominators | M9 |
| GET | `/v1/marketing/brands/:brandId/reports` | Authorized narrative/campaign comparisons | M9 |
| POST | `/v1/marketing/brands/:brandId/feedback` | Quality/outcome feedback linked to revision/run/post | M4/M9 |

Shared integration routes own OAuth/webhooks. Publishing state is service-controlled; callers cannot set `published` or submit an unverified external receipt.

## 15. Integrations, MCP, and optional OpenClaw

### 15.1 Connector architecture

Connectors implement public domain ports and declare supported reads/actions, schemas, scopes, pagination, rate limits, credential lifecycle, error classification, and receipt/reconciliation behavior. MCP is an optional standard surface over these capabilities, not mandatory plumbing for every internal function call.

| Integration | First use | Ownership/constraints |
|---|---|---|
| Curated calendar JSON | M0 | Verified market/year occurrences; no employee calendar access required |
| Keycloak | P1 | OIDC/account lifecycle; application authorization remains Blossom-owned |
| Model API | M0 | Server credentials, evaluated model, tenant budgets, no privileged instructions from sources |
| Research/search | M6 | Provider selected by pilot; approved sources, SSRF checks, source/date/expiry tracking |
| S3 | Documents/media | Private objects, authorized URL lifetime, hashes, version/retention |
| Instagram | M8 | Official API, current professional-account path/permissions, approved package, receipts |
| CRM/ERP | P4/P6 by demand | Client-specific mapping, incremental sync, minimum scoped reads/writes |
| E-signature/banking | P6 after discovery | Separate domain authority and reviewed external-effect requirements |
| OpenClaw | Optional | Explicit channel/operator use case and isolated trust boundary |

The MCP architecture defines hosts/clients/servers exposing tools, resources, and prompts. Blossom's gateway wraps those interfaces with its own permissions, identity, data classification, usage, and audit. A server's claim to be an MCP server does not make it trusted. See [MCP architecture](https://modelcontextprotocol.io/docs/learn/architecture).

Review/register allowlisted MCP servers and supported protocol versions. Use supported authenticated remote transport or supervised local transport appropriate to the deployment. Validate discovery/arguments/results, bound resources/time, pin versions, and keep credentials server-side. Do not accept arbitrary tenant-uploaded server commands. Raw database MCP servers with unrestricted SQL are not production agent capabilities.

### 15.2 External credentials and effects

Store secret references or envelope-encrypted credentials with owner, provider scopes, expiry, rotation key version, and revocation state. API keys are hashed and displayed once. Avoid secrets in snapshots, prompts, graph state, logs, queue payloads, browser bundles, images, or Factory memory.

Connection callbacks resolve owning tenant from validated OAuth state. Inbound webhooks validate signatures, deduplicate event IDs, and queue minimal work. Outgoing effects resolve the owned connection from a permitted resource, persist effect identity before sending, and retain provider IDs/receipts afterward.

For Instagram, the implementation must select and document its supported official login path and current account/review requirements. Allow the provider to fetch media only through a deliberately scoped delivery mechanism with adequate expiry; do not make the private source bucket public. Recheck ownership, approval, evidence/offer validity, account eligibility, pause/cancel state, and duplicate identity at actual publication time.

### 15.3 OpenClaw operating model

OpenClaw is optional and outside the first product release. Its gateway can receive channel requests and maintain operator sessions; Blossom remains the authority for commands, outcomes, and permissions. Its documented trust model assumes one trusted boundary per gateway, not hostile unrelated tenants sharing an agent/gateway. Use isolated gateways, credentials, storage, and execution environments for separate client trust boundaries. See [OpenClaw architecture](https://docs.openclaw.ai/concepts/architecture) and [security](https://docs.openclaw.ai/gateway/security).

Initial integration pattern:

1. Receive a channel request in an approved isolated host.
2. Resolve channel identity to a verified Blossom human/service identity. Require account linking where necessary.
3. Submit a known demand/task through a scoped authenticated Blossom API or approved MCP capability.
4. Blossom admits and executes the workflow, including required human review.
5. Return only authorized result references or a safe summary to the originating permitted channel.

A shared gateway service credential must not impersonate any channel sender as any tenant user. Use verified mapping, narrowed delegated scopes, and audit. Channel approval UI is only an ingress for the same exact-subject approval API; it is not a second approval ledger.

Do not make a long OpenClaw operator loop and a LangGraph workflow both own the same business execution. The bridge requests/observes a Blossom run. If a later isolated operator runtime is justified, it must satisfy the same runtime/task/tool conformance requirements and avoid duplicate recovery/effect ownership.

## 16. Infrastructure, environments, configuration, and commands

### 16.1 Deployment baseline

| Component | Development | Initial production |
|---|---|---|
| Web | Next.js dev server | Vercel |
| API | NestJS process or Docker | Render Docker web service |
| AI/outbox/scheduler workers | Separate processes/Compose services | Render background workers |
| Integration/render workers | Enable when their milestone starts | Separate worker pools |
| Application database | PostgreSQL with persistent dev volume | Managed PostgreSQL with tested backup/restore |
| Queue store | Dedicated Redis instance | Persistent compatible managed queue store, no eviction |
| Cache | Optional separate instance | Separate instance/policy; never evict queued work |
| Keycloak | Dev realm, separate identity database/role | Operated production container, TLS, its own database lifecycle |
| Documents/media | Storage adapter mock; isolated test bucket when needed | Private S3 bucket, scoped delivery and retention |
| Telemetry | Local collector/logs | Managed or operated telemetry backend with approved retention |
| OpenClaw | Disabled unless testing its specific adapter | Separate approved trust-boundary deployment |

Keep API/workers/database/queue close by region. Bound the total database connection count across all replicas. Vercel hosts the web layer; continuously running workers run on conventional worker services. Docker Compose is for local development, not the production autoscaling design.

Render Key Value is Redis-compatible; its underlying engine and supported settings must be checked for the selected service. Verify the pinned BullMQ features, persistence, and queue policy instead of assuming every Redis-branded endpoint is interchangeable. See [Render workers](https://render.com/docs/background-workers) and [Key Value](https://render.com/docs/key-value).

### 16.2 Environment contract

P0/P1 generate a documented `.env.example`; do not commit real credentials. Validate environment fields at startup by process/feature. Disabled optional integrations do not require credentials.

```dotenv
NODE_ENV=development
APP_PUBLIC_ORIGIN=http://localhost:3000
API_INTERNAL_URL=http://localhost:4000
PORT=4000
DATABASE_URL=postgresql://blossom_dev:local_dev_only@localhost:5432/blossom_dev
KEYCLOAK_DATABASE_URL=postgresql://identity_dev:local_dev_only@localhost:5432/identity_dev
REDIS_QUEUE_URL=redis://localhost:6379
REDIS_CACHE_URL=
OIDC_ISSUER_URL=http://localhost:8080/realms/blossom
OIDC_CLIENT_ID=blossom-web
OIDC_CLIENT_SECRET=replace_in_local_environment
OIDC_REDIRECT_URI=http://localhost:3000/api/blossom/auth/callback
SESSION_COOKIE_NAME=blossom_session
SESSION_ENCRYPTION_KEY=generate_for_environment
INTEGRATION_ENCRYPTION_KEY=generate_for_environment
ENCRYPTION_KEY_VERSION=dev-1
MODEL_PROVIDER=openai
MODEL_NAME=select_and_pin_after_evaluation
OPENAI_API_KEY=set_only_in_server_environment
MAX_MODEL_CALL_INPUT_TOKENS=12000
MAX_MODEL_CALL_OUTPUT_TOKENS=4000
MAX_RUN_REPAIR_ATTEMPTS=1
DEFAULT_TENANT_ACTIVE_RUN_LIMIT=2
DEFAULT_TENANT_PENDING_RUN_LIMIT=20
LANGGRAPH_ENABLED=false
LANGSMITH_TRACING=false
OTEL_EXPORTER_OTLP_ENDPOINT=
S3_REGION=
S3_BUCKET=
RESEARCH_PROVIDER=
INSTAGRAM_CLIENT_ID=
INSTAGRAM_CLIENT_SECRET=
INSTAGRAM_WEBHOOK_VERIFY_TOKEN=
OPENCLAW_ENABLED=false
```

The numeric limits are initial configurable planning values, not measured capacity. Whole-run budgets additionally limit cumulative calls, tools, and output. Server variables never use `NEXT_PUBLIC_*`. Prefer workload identities/host secret management where supported; do not put long-lived object-store keys in frontend configuration.

Within Compose, service hostnames differ from `localhost`; configure internal resolution while preserving the verified public OIDC issuer and callback. Do not disable TLS/issuer validation to bypass networking problems. Use separate dev, staging, and production credentials, databases, buckets, identity realms, queue stores, and model budgets.

### 16.3 Developer command contract

These commands become runnable when the specified packages/scripts exist. The README must not advertise them as tested until CI verifies them.

```bash
pnpm install --frozen-lockfile
pnpm prototype --brand examples/brand.json --calendar examples/calendar.json --month 2027-01
docker compose -f infrastructure/docker/compose.dev.yml up -d postgres redis keycloak
pnpm db:migrate
pnpm db:seed
pnpm dev
pnpm lint
pnpm typecheck
pnpm test
pnpm test:integration
pnpm test:e2e
pnpm eval:marketing
pnpm build
```

M0 needs only its small CLI package and model/fixture configuration. It does not require the hosted platform. Local seeding uses synthetic records and must refuse production targets. Database migrations run in one controlled job, not on every API/worker startup.

### 16.4 CI/CD and release procedure

1. Install pinned dependencies; check formatting, lint/import boundaries, and types.
2. Run relevant domain/API/isolation/recovery tests and offline evaluation gates.
3. Build affected applications and versioned immutable images/artifacts.
4. Run reviewed database migrations with separate credentials and compatibility evidence.
5. Deploy staging; exercise the critical business path with synthetic/pilot data.
6. Release a limited candidate under product approval and rollout policy; track operational/quality metrics.
7. Preserve prior application/prompt/workflow versions and document rollback/restart for in-flight runs.

Hooks may provide developer feedback but do not replace CI. Branch protection and required reviews prevent a Factory role from bypassing release gates. Hosting provider choice is an implementation baseline, not an assumption that accounts/plans have already been provisioned.

## 17. Growth, observability, costs, and operations

### 17.1 Scaling approach

| Growth pressure | First response | Later trigger |
|---|---|---|
| More concurrent users | Stateless API replicas, scoped caching, indexes/pagination | Dedicated services if ownership/scaling warrants them |
| More AI work | Per-tenant admission, bounded pools, provider token/rate budgets | Additional evaluated providers/isolated pools |
| More client data | Incremental imports, bounded jobs, retention, derived indexes | Partitioning/replicas/dedicated storage after measured limits |
| Heavy media | Separate rendering pool and resource limits | Specialized compute/video pipeline |
| One tenant dominates | Pending/active/cost caps and application fair admission | Higher isolated capacity by entitlement |
| Stronger isolation needs | Scoped repositories, credentials, networks, retention | Dedicated tenant deployment/database via routing adapter |
| Cross-domain workflows | Typed capabilities and versioned events | Service extraction/event platform if justified |

Application fair scheduling must be implemented; do not assume open-source BullMQ automatically includes commercial group/fairness features. Persist admission decisions/reservations and enforce budgets atomically. Prevent concurrency from exceeding tenant limits through distributed-safe leases/admission, not only an in-memory counter.

Planning load scenarios, to validate with realistic data and mocked model providers:

| Profile | Tenants | Concurrent sessions | Short API requests/second |
|---|---:|---:|---:|
| Pilot exercise | 10 | 20 | 5 |
| Launch exercise | 100 | 200 | 50 |
| Growth exercise | 1,000 | 2,000 | 250 |

These are hypotheses, not supported capacities. A starting latency target is p95 below 500 ms for ordinary reads and below one second for generation admission at the agreed launch workload. Model-job completion/queue wait require separate targets by task class and provider quota. Agree actual SLO/RPO/RTO before client launch and demonstrate them on the chosen infrastructure.

### 17.2 Observability

| Signal | Required dimensions/behavior |
|---|---|
| Request/job trace | Request, tenant, actor/service, run, step, correlation and safe resource IDs |
| API health | Latency percentiles, error classes, dependency readiness |
| Delivery health | Backlog count/age, outbox lag, expired leases, failed/dead-letter work |
| Workflow health | Pauses, validation failures, retries/repairs, recovery, cancellation latency |
| Model health | Actual input/output tokens, latency, provider errors, model/version, usage/cost |
| Connector health | Last sync/cursor, authentication expiry, webhook attempts, uncertain effects |
| Data health | Ingestion failures, stale sources, retrieval quality, index/storage growth |
| Business quality | Acceptance/editing time, unsupported claims, approved outcomes, feedback |
| Security/authorization | Denials/revocations, anomalous requests, support grants, integration changes |

Redact secrets and sensitive content. Avoid raw prompts/documents in routine logs. Optional LangSmith exports require approved tenant/data/retention policy and access control. Limit high-cardinality telemetry; detailed tenant/run drill-down belongs in authorized records/traces rather than unrestricted public metrics.

### 17.3 Cost and entitlements

`model cost = Σ(input tokens × input rate per token + output tokens × output rate per token + tool fees)`

Add hosting, database, queues, identity, telemetry, research, storage/transfer, rendering, and optional media generation. Record provider/rate date, actual usage, retries/failed calls, and cost per approved output. A cheap first draft can still be expensive after edits and retries.

Reserve estimated budget at admission, reconcile actual charges, release unused reservations, and account for calls completed after cancellation. Restrict model fallbacks to evaluated policy and record the actual provider/model used. Quota changes are audited. Product usage/billing records are separate from a future company's accounting ledger.

### 17.4 Operational and data-governance requirements

- Back up application and identity databases; provision appropriate recovery and exercise restoration. Back up/retain required objects and verify referenced assets after restore.
- Restore business records and workflow checkpoints consistently enough to reconcile interrupted work; recovery must not re-send uncertain external actions blindly.
- Define tenant-specific retention for source data, checkpoints, conversations, logs, traces, audit, exports, and deleted records. Support data export/erasure procedures and derived-index cleanup.
- Keep audit append-only to application actors; privileged administrative changes have their own operational trace. Do not claim tamper-proof audit without additional controls.
- Name owners for incidents, support grants, publishing pauses, key rotation, backups, and provider outages.
- Provide tenant/domain emergency pause controls; publication pause blocks new effects while receipts can still be reconciled.
- Exercise database/queue outage, provider failure, integration revocation, restore, and workflow-version rollback before the relevant release.
- Select region, contractual data handling, trace export, and retention with the business/client owner. A framework selection alone does not establish regulatory compliance.

## 18. Blossom Factory engineering system

### 18.1 ECC adaptation

The Factory uses ECC engineering assets through a selected coding harness. It does not run simply because an `agents/` directory exists. F0-01 audits the actual fork/installed version, discovery conventions, available roles, commands, hooks, rules, license, and tool permissions. Record the upstream commit and local changes.

The supplied fork reference is [LucasAlDev/ECC](https://github.com/LucasAlDev/ECC). Its exact current configuration was not available during this document's public-source review. Official upstream [ECC](https://github.com/affaan-m/ECC) and its [planner definition](https://github.com/affaan-m/ECC/blob/main/agents/planner.md) support the engineering/planning model; they do not prove a custom role is installed in the fork.

Keep canonical Blossom assets in `blossom-factory/`; harness adapters install/reference them according to the selected tool's supported discovery rules. Preserve existing ECC files until their behavior is understood. Do not replace the fork wholesale with the proposed product tree or mix Factory agent files with production definitions.

### 18.2 Engineering roles

These are responsibilities to map to existing ECC roles or implement as custom Factory roles. Names are proposed unless verified in F0-01.

| Role | Deliverable | Authority boundary |
|---|---|---|
| Planner | Work-package implementation plan, dependencies, steps, acceptance checks | Does not silently expand scope or mark unbuilt routes complete |
| Platform architect | Module/tenant/runtime boundaries and ADRs | Changes need explicit rationale/contract review |
| Backend engineer | Use cases, controllers, queues, authorization | Writes owned services; no cross-domain private SQL shortcuts |
| Frontend engineer | Forms, views, safe API integration, UX states | No browser secrets or client-only security |
| Database engineer | Schema, migration, RLS, indexes, data/recovery design | Separate migration access; no casual destructive production action |
| Workflow engineer | State/schema, graph, model nodes, runtime recovery | Tools/approvals are enforced outside prompts |
| Integration/MCP engineer | Scoped connectors and effect/reconciliation contracts | No unreviewed server commands/credential exposure |
| Test/evaluation engineer | Business, isolation, recovery, and quality evidence | Synthetic fixtures; bounded real-provider calls |
| Security reviewer | Tenancy, identity, injection, secrets, network/effect review | Findings must be resolved or explicitly accepted by accountable owner |
| Code reviewer | Correctness, maintainability, contract ownership | Reviews a concrete diff and test evidence |
| Release/operator role | Staging candidate, migration/rollback/runbooks | Production promotion follows release policy |

### 18.3 Factory skills and rules to establish

| Canonical skill/rule | Contents |
|---|---|
| `blossom-architecture` | This architecture, import rules, ownership, modular deployment |
| `requirements-to-work-package` | Traceability, task decomposition, dependency and acceptance templates |
| `multi-tenancy` | Trusted scope, RLS, worker/checkpoint/storage/cache isolation |
| `api-contracts` | Runtime schemas, OpenAPI, status/idempotency/version conventions |
| `langgraph-workflows` | State, checkpoints, pause/resume, compatible version recovery, effect boundaries |
| `langchain-components` | Model/tool adapters, structured outputs, budgets, evaluated fallbacks |
| `mcp-integrations` | Approved tools/resources, auth, source validation, connector policy |
| `business-evidence` | Provenance, eligibility, revision freshness, factual/public-use review |
| `security-and-secrets` | OIDC/CSRF, credential storage, injection/network boundaries |
| `evaluation-and-observability` | Rubrics, fixtures, safe traces, usage, release evidence |
| `delivery-and-review` | Branch/worktree isolation, contracts before implementation, CI and PR evidence |

Factory memory stores confirmed engineering patterns, ADR links, sanitized failure examples, and verified commands. It never silently rewrites business policy or promotes production prompts from observed feedback. Proposed improvements become reviewed work packages.

### 18.4 Engineering execution rules

1. Read the root `AGENTS.md`, applicable directory rules, the bounded work package, referenced README sections, and current code.
2. Resolve the existing repository state and implemented contracts before planning changes.
3. Identify upstream contracts, migrations, domain owners, and accepted dependencies. One accountable owner owns each shared artifact.
4. Create the implementation plan with exact paths, failure cases, evidence, and scope exclusions.
5. Implement in an isolated branch/worktree or the repository's approved equivalent.
6. Delegate independent work only when the active harness/workflow permits it and boundaries are explicit. Avoid two agents editing the same migration, shared schema, or entry point concurrently.
7. Validate schemas/public contracts, business rules, authorization, failure behavior, and relevant output quality.
8. Obtain concrete code/security/evaluation review proportional to the package.
9. Produce a reviewable PR/diff, checks, migration notes, runbook updates, and remaining limitations.
10. Merge/release according to established repository controls. Engineering autonomy cannot bypass production/business approval.

An agent-generated plan is not completion evidence. Update package status to `planned`, `in_progress`, `review`, `done`, or `blocked` using actual artifacts. Document blockers and assumptions without fabricating test results. Reuse validated implementation rather than generating a parallel version in another domain.

### 18.5 Definition of done for a Factory work package

The specified behavior works; contracts and ownership are documented; inputs and permissions are validated; failures/retries/side effects follow their policy; meaningful checks pass; any required migration is reviewed; related evaluations have recorded outcomes; a reviewer can inspect the diff and reproduce verification. Prototype shortcuts are explicitly tagged and removed before shared deployment.

Do not generate a control panel, autonomous IDE, or agent marketplace merely to start the Factory. F0/F1 deliver repeatable engineering workflows. F2 improves them using observed, sanitized delivery evidence after the first release.

## 19. Implementation phases and assignable work packages

### 19.1 Phase map

**F** phases establish the engineering Factory. **P** phases establish the platform. **M** phases deliver Marketing. They are linked workstreams, not three applications to finish sequentially. This backlog supersedes the attachments' issue numbering; the preserved business milestones remain M0–M9 and P0–P6.

| Phase | Outcome | Gate |
|---|---|---|
| F0 — Factory/bootstrap audit | Verified ECC/harness configuration and planner handoff | Actual discovery paths, roles, tools, provenance recorded |
| F1 — Engineering repeatability | Blossom skills/rules and review/CI procedure | One bounded package travels from requirement to reviewable evidence |
| F2 — Factory improvement | Evaluated engineering-pattern updates | Sanitized delivery feedback improves a measured engineering outcome |
| P0 — Architecture/scaffold | Versioned contracts, workspace/package conventions, ADRs | Pinned install/typecheck and agreed pilot inputs |
| P1 — Identity/data | Sessions, tenants, grants, versioned catalog/evidence/calendar, OpenAPI | Actual-role SQL/API isolation and identity failure cases pass |
| P2 — Execution/review | Runs, jobs/outbox, tools/models/budgets, approvals, audit | Duplicate/crash/revocation/review scenarios preserve correct behavior |
| P3 — Internal release | Demand tracking, deployed staff Marketing, operations | Brand → plan/draft → edit → review → export with usable output |
| P4 — Client readiness | Delegation, scoped APIs/connectors, ingestion, webhooks, capacity/recovery | Limited external client rollout with isolation and operations evidence |
| P5 — Advanced execution | Versioned graph runtime, registry, scheduler, isolated worker pools | Authorized pause/restart/resume and bounded typed collaboration |
| P6 — New business domains | Owned requirements and first controlled domain workflows | Domain owner approves data/action/review boundaries and pilot outcome |
| M0 | One brand/month local proof | Useful inspected output and known corrections |
| M1 | Pure Time/Value contracts | Deterministic source/date/eligibility behavior |
| M2 | Local UI/API proof | Marketer operates forms and understands output/findings |
| M3 | Persistent authenticated monthly text | Scoped durable runs, edits, review, history |
| M4 | Yearly outline, month expansion, export | Twelve unique briefs and evaluated selected-month drafts |
| M5 | Durable planning and rolling refresh | Checkpoint recovery, review resume, stale-input handling |
| M6 | Reviewed current research | Dated applicable sources/signals and safe failure fallback |
| M7 | Static posts/carousels | Designer-reviewed rendered package without overflow |
| M8 | Scheduled official publication | Eligible test account, exact-package approval, receipt/recovery |
| M9 | Measurement and reviewed improvement | Defined metrics and evaluated prompt/rule changes |

P5/M5 can proceed after the internal release without waiting for all client-commercial features. Internal M6–M9 need their shared security/integration building blocks, but unrelated external-client use additionally requires the P4 launch gate.

### 19.2 Work-package conventions

One row is a planner intake, not necessarily one PR. Split a row into smaller implementation issues when it spans multiple independent contracts. Preserve its acceptance criteria and traceability. The planner adds exact files/functions, schema details, edge cases, size/risk, and verification commands after inspecting the current repository.

Owners below: **PL** platform/architecture; **BE** backend; **DB** database; **FE** frontend; **AI** model/workflow; **INT** integrations; **QA** test/evaluation; **SEC** security review; **PO** accountable business/product owner. Map engineering roles to verified Factory agents; PO decisions and required human business review remain accountable human responsibilities.

Paths are implementation targets under the repository root. Dependencies are accepted packages, not merely generated plans. Shared schema/migration changes have a single owner.

### 19.3 Factory packages

| ID | Owner | Files/boundary and deliverable | Depends on | Acceptance |
|---|---|---|---|---|
| F0-01 | PL | `blossom-factory/upstream/provenance.md`, `adapters/`; inspect ECC fork/harness, licenses, installed roles/commands/hooks | None | Installed capabilities distinguished from proposed custom roles; version/paths/tool permissions recorded; no production dependency introduced |
| F0-02 | PL | Root `AGENTS.md`, Factory templates/rules; bounded planning and scope/ownership rules | F0-01 | Selected harness loads applicable guidance; one sample package yields explicit dependencies, files, checks, assumptions |
| F1-01 | PL/AI/SEC | Factory architecture, tenancy, workflow, MCP, evidence, security skills | F0-02, P0-02 | Guidance references current contracts; review catches seeded tenancy/effect-boundary mistakes |
| F1-02 | PL/QA | `.github/workflows/`, Factory delivery/PR templates; reproducible checks and review mapping | F0-02, P0-01 | A sample diff runs relevant checks; reviewers receive actual evidence and limitations |
| F2-01 | PL/QA | `blossom-factory/evaluations/`, sanitized memory updates | P3-03 | Measured engineering improvement without importing client secrets/data or changing business policy implicitly |

### 19.4 Platform foundation packages

| ID | Owner | Files/boundary and deliverable | Depends on | Acceptance |
|---|---|---|---|---|
| P0-01 | PL | Root workspace/scripts, `packages/config`, app entry scaffolds; compatible pinned Node/TS/module/test setup | F0-01 | Clean pinned install, import smoke, typecheck/build scaffolds work; no optional infrastructure needed by M0 |
| P0-02 | BE/AI/PO | `docs/requirements`, `packages/api-contracts`, `packages/primitives`; frozen pilot/task/output conventions and glossary | None | Required fields, missing-input behavior, output trust boundary, limits and review rubric agreed |
| P0-03 | PL | `docs/adr`, architecture ownership/version notes | P0-02, F0-01 | Tenant/session, module, revision, runtime, delivery, deployment decisions recorded with rationale |
| P1-01 | DB | `packages/database/schema`, migrations, transaction scope, runtime/admin roles and RLS | P0-01, P0-02 | Runtime-role cross-tenant read/insert/link attempts fail; pooled context does not leak; reviewed migration applies |
| P1-02 | BE/PL | Identity/session module, Keycloak config, same-origin proxy/auth callbacks | P1-01, P0-03 | Login/logout/expiry/CSRF/state/nonce/redirect/refresh failure and cookie forwarding verified |
| P1-03 | BE/SEC | Workspace/membership/resource-grant services and customer registry | P1-01, P1-02 | Scoped reads/actions deny other tenant/brand; membership revocation prevents delayed authority |
| P1-04 | BE/DB | Catalog, evidence, calendar ports/repositories/versioned input APIs | P1-03 | Exact versions/source validity survive edits; invalid/expired/private-use input is ineligible |
| P1-05 | BE/FE | OpenAPI generation, public runtime schemas, web API-client types | P1-03, P0-02 | Implemented routes match schemas; invalid inputs fail server validation; unbuilt routes remain documented backlog |
| P2-01 | BE/DB/PL | Execution registry, jobs, input snapshots, leases, outbox/inbox, dispatcher/BullMQ | P1-01, P1-03 | Commit/enqueue gap, duplicate delivery, expired lease, API/worker crash and queue-loss recovery exercised |
| P2-02 | AI/BE | Runtime/simple adapter, model gateway, tool policy, usage reservations/limits | P2-01, P0-02 | Unsupported task/tool rejected; provider failures bounded; cumulative cost and atomic tenant limits visible |
| P2-03 | BE/SEC | Subject registry, immutable revision approval ledger, human tasks | P1-04, P1-03 | Approval applies only to eligible exact revision; forged approval/new edit cannot inherit it; domain policy enforced |
| P2-04 | PL/BE/FE | Safe logs/traces/audit, run/history/review API and UI contracts | P2-01, P2-02, P1-05, M2-01 | Authorized user traces request → run → output; secret/private content redacted; safe failures and usage shown |

### 19.5 Internal release, clients, and advanced platform packages

| ID | Owner | Files/boundary and deliverable | Depends on | Acceptance |
|---|---|---|---|---|
| P3-01 | PL | Docker images, Vercel/Render staging, environment schemas, controlled migration job | P1-02, P2-01, F1-02 | Staging services start with isolated credentials and known database/queue/identity network configuration |
| P3-02 | BE/FE | `platform/demands`, demand API/UI, supported routing and run/outcome links | M3-03, M3-04, P1-03 | Known Marketing demand executes through the same admission service; unknown/missing-input demand remains triaged; requester sees authorized outcome |
| P3-04 | PL/BE | `docs/runbooks`; backup/restore, key rotation, run recovery, pause/support ownership | P3-01, M3-05 | Operators perform documented staff-release restore/recovery; in-flight effects are reconciled |
| P3-03 | QA/PO/PL | Staff pilot report, onboarding, internal release evidence | M4-04, P3-01, P3-02, P3-04 | Complete authenticated text path and section 20 staff gates pass; operational/quality/cost limitations recorded |
| P4-01 | BE/SEC | Delegation, service accounts/API keys, access revocation and limits administration | P3-03 | Client/support identities stay within assigned scope; expired/revoked grants/keys block delayed actions |
| P4-02 | INT/BE | Knowledge source registration, file/import pipeline, S3 adapter, incremental cursor/deletion contracts | P1-04, P2-01, P4-01 | One approved source imports safely; invalid rows isolated; resumable batches retain lineage; deletion/expiry reaches derivatives |
| P4-03 | INT/BE | Signed client webhooks, validated destinations, delivery/retry history; connector framework/MCP pilot | P4-01, P2-01 | Duplicate/failing/unsafe destinations handled; only scoped allowlisted capabilities exposed |
| P4-04 | QA/PL/SEC | Isolation/load/fairness/restore and incident exercises | P4-01, P4-02, P4-03 | Measured report states hardware/data/workload/limits; cross-tenant surfaces and overload/recovery gates pass |
| P4-05 | PO/PL | Limited external client onboarding/support/data-use plan | P4-04 | Client owns approved data/actions; isolation, retention, quotas, support and restore processes demonstrated |
| P5-01 | AI/BE/DB | LangGraph runtime, authorized checkpoint adapter, immutable workflow/agent registry and deployment APIs | P2-02, P2-03, M4-04 | Same contracts execute after restart; all checkpoint operations enforce tenant ownership; incompatible version resume rejected/migrated explicitly |
| P5-02 | BE/PL | Durable schedules, due-occurrence IDs, admission fairness, provider-wide limits | P2-01, P5-01 | Missed/duplicate scheduler delivery is safe; timezone cases and multi-tenant fairness verified |
| P5-03 | PL/INT | Separate integration/render worker pools and resource/network/storage controls | P2-01, P4-02 | Heavy rendering/import cannot starve API/AI; assets remain privately scoped; queue recovery preserved |
| P5-04 | AI/BE | Typed specialist tasks/child runs with shared budget, depth/timeout/cancel controls | P5-01, P6-04 | One approved cross-domain collaboration uses least necessary data; no cyclic/unbounded delegation |
| P6-01 | PO/BE | Contracts process/data/permission/approval/API/evaluation specification | P3-03 | Legal/domain owner agrees sources, jurisdiction, review/signature boundary and one bounded pilot workflow |
| P6-02 | PO/BE/DB | Finance process/mapping/exact-decimal/calculation/reconciliation specification | P3-03 | Finance owner agrees source, tolerances, deterministic rules, review/payment boundary and one pilot workflow |
| P6-03 | PO/BE | Operations process spec, org units, exception/notification/schedule permissions | P3-03 | Process owner defines steps, outcome, failure/human routing and permitted external actions |
| P6-04 | Domain team/QA | First additional domain package/agent/API and pilot | P5-01; one approved P6-01/P6-02/P6-03 spec | Owned workflow passes domain and platform gates; unrelated-client deployment also requires P4-05 |

P4-02 can serve later internal research as soon as its prerequisites are accepted. P5-03 is shared worker infrastructure and has one owner; Marketing integrates with it. P6 is a discovery gate followed by explicit implementation packages derived from the approved specification, not an instruction to fabricate complete department systems.

### 19.6 Marketing packages

| ID | Owner | Files/boundary and deliverable | Depends on | Acceptance |
|---|---|---|---|---|
| M0-01 | PO/AI | `examples/brand.json`, `calendar.json`, pilot rubric; one brand/audience/offer, three claims, reviewed dates | P0-02 | Every real pilot fact has an owner/source; required context and public-use limits explicit |
| M0-02 | AI | `apps/prototype/src/cli.ts`; bounded structured monthly generation and readable text/JSON | M0-01, P0-01 | Reproducible local command handles valid/missing inputs and records model usage; no platform deployment needed |
| M0-03 | PO/QA | Pilot alternatives, corrections, editing time and usefulness report | M0-02 | Two reviewers agree rubric and identify actual factual/relevance failures before expanding |
| M0-04 | INT/PO | Instagram feasibility note for future account owner/login/access/review path | None | Later external dependency documented; no publication implemented or assumed approved |
| M1-01 | AI/BE | Marketing input/output/context schemas and independent public ports | M0-03 | Required fields, eligible reference sets, server-owned metadata, missing-data behavior validated |
| M1-02 | AI/BE | `marketing/skills/time`; verified occurrence/season/window/evergreen logic | M1-01 | Wrong-market, movable-date, local-season and no-event cases behave deterministically |
| M1-03 | AI/PO | `marketing/skills/value`; benefit/evidence/claim eligibility and visible rubric | M1-01 | Unsupported/expired/private-only claim use prevented; unknown comparisons and scores explained |
| M1-04 | AI/QA | Versioned prompts, bounded structured-output validation/repair, early evaluations | M1-02, M1-03 | Candidate references/wording checked; malicious brief cannot expand tool policy; known cases reproducible |
| M2-01 | FE | Local brand/month forms and readable preview/source findings with fixtures | M1-01 | Marketer enters inputs without editing JSON; missing fields and useful rationale visible |
| M2-02 | BE | Thin local NestJS adapter to existing capabilities; bounded mock/dev generation | M1-04, P0-01 | Server validation works; domain rules remain outside controllers; local-only shortcuts labeled |
| M2-03 | FE/BE | Fixed-target web proxy, safe errors, usability pass | M2-01, M2-02 | No model keys in browser; user corrects a monthly draft through the local interface |
| M3-01 | BE/DB | Brand/profile/audience/claim/assignment persistence and APIs | P1-04, M1-01 | Stable scoped records and immutable revisions reload; referenced tenant/brand/version ownership enforced |
| M3-02 | FE | Authenticated brand onboarding, Time/Value views, eligible selectors | M3-01, P1-02, M2-03 | Authorized staff completes inputs and sees ineligible/missing evidence without hidden prompt workarounds |
| M3-03 | AI/BE | Asynchronous monthly task, immutable snapshot, run/output materialization | M3-01, M1-04, P2-01, P2-02 | API/worker restart and duplicate requests preserve one coherent result identity and recorded cost |
| M3-04 | FE/BE | Draft editor, revision guards, exact review, safe run/history/cost views | M3-03, M3-02, P2-03, P2-04 | Concurrent edits conflict; new edit has no approval; source/usage/errors visible |
| M3-05 | QA/SEC | Two-tenant/brand/revocation, job failure, revision and export-access checks | M3-04, P1-01, P2-01 | Unauthorized inputs/results/reviews fail; eligible crash/retry/cancel path demonstrated |
| M4-01 | AI/PO | Annual-outline task and twelve-month schema/coherence checks | M3-04 | Exactly twelve unique months, supported Time/Value references, realistic lead times |
| M4-02 | FE/BE/AI | Annual planner, immutable plan edits/review, selected-month expansion | M4-01 | Parent revision preserved; four to eight drafts align with reviewed strategy and current eligibility |
| M4-03 | FE/BE/AI | Allowed-section regeneration, compare, labeled Markdown/JSON export, feedback | M4-02 | Selective edit creates new revision; exact approved export excludes unreviewed replacements |
| M4-04 | QA/PO | 20–30-case evaluation and staging staff journey | M4-03, M3-05, P3-01 | Section 20 text gates and editing-time/cost report recorded; no false capacity or quality claims |
| M5-01 | AI/BE | Marketing graph/state implementation using shared runtime adapter | M4-04, P5-01 | Same contracts and quality hold; persisted completed steps survive restart |
| M5-02 | AI/BE/FE | Human pause/resume, safe step timeline and selected failed-step recovery | M5-01, P2-03 | Paused work releases capacity; forged/wrong-revision/other-tenant resume denied |
| M5-03 | AI/BE | Source dependency flags, rolling refresh and bounded optional bulk-year drafts | M5-02, P5-02 | Changed/withdrawn inputs flag unpublished results; refresh is selective/idempotent/budgeted |
| M6-01 | PO/AI/INT | Sector questions, source/use/expiry policy and research-provider evaluation | M4-04 | Approved source categories and dated comparison/observation requirements agreed |
| M6-02 | INT/AI | Scoped search/fetch and source persistence, reviewed graph research task | M6-01, P4-02, P2-02, M5-01 | Network/size limits enforced; every usable signal has source, observation/publication date, market and expiry |
| M6-03 | FE/QA/PO | Signal accept/reject/expiry UI and malicious/stale/conflicting-source evaluation | M6-02 | Only eligible reviewed signals inform releasable strategy; provider absence yields approved-data fallback or visible gap |
| M7-01 | Designer/PO | One approved post and small carousel template with dimensions/text limits | M4-04 | Typography, spacing, contrast, CTA and brand asset use reviewed |
| M7-02 | AI/BE | Structured creative specification linked to exact reviewed content/template | M7-01 | Ordered slides, bounded text, caption/alt text, source/media ownership validated |
| M7-03 | INT/PL/FE | Isolated render worker, fonts/assets, S3 versions and preview/revision UI | M7-02, P5-03 | Representative renders have no clipping/overflow; uploads/previews scoped; failures recoverable |
| M7-04 | BE/FE | Publication-package subject and designer/strategist review | M7-03, P2-03 | Exact caption/text/ordered media/material settings bound; changed material requires new approval |
| M8-01 | INT/PO | Current API eligibility/login review, shared OAuth credential lifecycle | M0-04, P4-01 | Eligible account connects; refresh/expiry/revocation/disconnect behavior verified; external-client requirements recorded |
| M8-02 | INT/BE | Official media preparation/publish/status adapter and receipts | M8-01, M7-04 | Authorized reviewed test media publishes; exact external identity/receipt persisted |
| M8-03 | BE/FE/QA | Package schedule, timezone, pause/cancel, duplicate guard and uncertain-outcome reconciliation | M8-02, P5-02, M5-03 | Execution rechecks grants/approval/sources/account; simulated timeout does not cause blind duplicate publish |
| M8-04 | PO/QA/PL | Small publication pilot and operator guide | M8-03 | Publisher sees accurate state, can pause, and performs recovery; external-client use additionally gated by P4-05 |
| M9-01 | PO/Analyst | Metric/qualified-lead/denominator/window/attribution definitions | M4-03 | Editorial usefulness and business outcome measures separated; manual early feedback usable |
| M9-02 | INT/FE/Analyst | Metric sync linked to narrative/package/post/offer and comparison reports | M9-01, M8-04 | Source/API version, window, completeness, missing data and consistent denominators visible |
| M9-03 | AI/QA/PO | Reviewed feedback-driven prompt/rule experiment and offline regression | M9-02, M4-04 | Candidate improves agreed metric without factual/permission regressions; causal limitations stated |

### 19.7 Scheduling and parallel delivery

Begin fixtures/business review immediately. The Factory audit and package scaffold can proceed while M0 inputs are prepared. After M1 schemas stabilize, UI and shared platform development can proceed through independent contracts. A small learning team might spend one to two weeks on M0/M1 and four to eight further weeks on M2–M4 plus P1–P3; these are inherited planning ranges, not commitments. Re-estimate after repository audit, skills, available hours, provider access, and pilot quality are known.

Parallelize only independent work. Frontend uses agreed fixtures/generated API types while backend implements the contract. Database migration/schema changes are serialized by their owner. Graph development depends on accepted domain/input/runtime contracts. Design can prepare templates before rendering implementation. Provider account/app review may be an external scheduling dependency.

Every package handoff includes relevant invariants, exact contract versions, fixtures, scope, dependencies, and acceptance evidence. Avoid feeding the planner unrelated raw repository output or every future domain requirement when one bounded task is being implemented.

## 20. Evaluation, verification, and release gates

### 20.1 Verification by layer

| Layer | Meaningful checks |
|---|---|
| Domain logic | Dates/market/season, claim/evidence eligibility, exact-decimal rules, revision transitions |
| Schema/API | Runtime validation, allowed selectors, status/errors, idempotency and expected-version behavior |
| Identity/access | OIDC callback/session/CSRF failures, actual grants, nested ownership, revoked service authority |
| Database | Runtime-role RLS, cross-tenant foreign keys, transaction pooling, migrations/indexes |
| Execution | Commit/enqueue gap, duplicate delivery, worker lease crash, partial steps, cost ceilings, cancellation |
| Graph | All checkpoint operations scoped, interrupt/restart replay, compatible version resume, artifact reconciliation |
| Integration | Connection ownership/scopes/expiry, signatures, unsafe URL denial, external receipt ambiguity |
| UI | Brand → monthly/annual result → edit → review → exact export; conflict/stale/failure states |
| Rendering/publication | Text overflow, media ownership/package version, paused/revoked/stale input, timeout/reconciliation |
| Capacity/operations | Dataset/workload-defined load, fair admission, queue outage, restore, runbooks |
| AI output | Human factual/usefulness rubric, structured output, bad references, injection, missing/conflicting facts |

Use mocked providers for most functional/load tests and a small bounded real-provider pilot. Test behavior and invariants, not incidental implementation details. Model graders may assist triage; they are not sole proof of factual accuracy, authorization, or review quality.

### 20.2 Marketing evaluation set

By M4, maintain about 20–30 sanitized cases covering normal campaigns, irrelevant holidays, movable dates, nonstandard local seasons, absent market, no relevant event, unsupported/expired claims, unavailable offer, private-only testimonial, conflicting evidence, changed inputs, fabricated/wrong reference, prompt injection, model wording that misstates valid evidence, concurrent revision edits, and two-client isolation.

Evaluation manifests record dataset/rubric/prompt/workflow/model/schema versions, parameters, usage, failure findings, reviewer outcomes, editing time, and date. Keep real customer data out of general fixtures and Factory memory.

### 20.3 Staff text release gate: P3-03

- Selected dates/market/seasons match verified occurrences and configuration.
- Releasable factual claims are supported by current approved evidence and checked by an authorized reviewer; unresolved blocking findings prevent approval.
- Annual outlines cover twelve unique months with coherent variation and reasonable preparation windows.
- Every output retains immutable input/source versions, selected overrides, model/prompt/workflow version, run and revision.
- The proposed editorial target is at least **80% accepted with minor edits** under a rubric agreed before evaluation; report actual results and editing time. This is a project target, not a proven result or external standard.
- Other tenants/brands cannot read, generate with, approve, or export protected data, including delayed work after revocation.
- Concurrent edits receive explicit conflicts. New edits do not inherit prior approval.
- Duplicate/crashed/failed work has visible state, bounded recovery/cancellation, and actual usage accounting.
- Authorized staff completes the deployed brand → plan/month draft → edit → review → export journey.
- Required migrations, backups/recovery, safe logs, ownership, and support documentation are ready.

### 20.4 Additional gates

| Gate | Additional evidence |
|---|---|
| External client launch | P4 delegation/API/connector isolation; ingestion/data-use/retention; fair limits/load report; restore/support onboarding |
| Durable graph release | Tenant-safe persistence; owned resume; same quality/contracts; replay-safe effects; workflow version compatibility |
| Research release | Dated applicable reviewed signals; safe source/network boundaries; stale/conflicting/unavailable source behavior |
| Creative release | Reviewed templates/assets; repeatable no-overflow renders; exact package revision and ownership |
| Publication release | Current official account/API eligibility; exact-package approval; execution rechecks; pause, receipts and reconciliation drill |
| New domain release | Accountable owner's process/data/action spec; domain evaluation; deterministic critical rules; applicable client gate |
| Factory improvement | Recorded engineering outcome; sanitized memory; no implicit expansion of product authority |

Track editorial acceptance, unsupported-claim rate, time to approved content, cost per approved output, run recovery/backlog age, defined audience response, qualified demand, attributed conversion, and relevant retention/repeat purchase. State denominators, attribution method, data coverage, and comparison limitations.

Relevant checks must pass before release. Repeat/broaden them when changes introduce new failure or risk; do not turn every low-impact wording/UI edit into an unrelated testing program.

## 21. Planner handoff templates and copyable prompts

### 21.1 What to give the planner for one package

| Context item | Required content |
|---|---|
| Common architecture | Source/status, ownership/import rules, tenant/auth invariants, execution/revision policy |
| Selected requirement | One package ID, business outcome, applicable README sections and requirement IDs |
| Current repository | Actual files, installed ECC/harness roles, package versions, existing contracts/tests |
| Dependencies | Accepted package evidence and exact contract/schema versions; blockers explicitly named |
| Boundary | Allowed directories, owned module/tables, public interfaces, prohibited scope expansion |
| Examples | Sanitized valid/invalid inputs, expected output/findings, failure/recovery scenarios |
| Acceptance | Observable business behavior, authorization, quality, failure and migration checks |
| Delivery | Plan, implementation issues, diff/PR, evaluation/test evidence, runbook/ADR updates |

Do not paste every private client record into the Factory. Provide synthetic fixtures or permitted redacted schemas. Business policy decisions stay explicit. The planner uses current code to refine file-level details; this README provides the initial target architecture.

### 21.2 Concrete work-package template

Example for **P2-01**; refine exact files/migration names after repository inspection:

```yaml
id: P2-01
title: Durable run admission and recoverable queue delivery
status: planned
owner_role: backend_engineer
review_roles: [database_engineer, security_reviewer, test_engineer]
requirements: [SYS-08, SYS-09, SYS-12]
readme_sections: [7, 8, 9, 10, 12, 17, 20]
depends_on: [P1-01, P1-03]
business_outcome: Accepted work stays visible and recoverable across dispatch or worker failure.
allowed_paths:
  - packages/platform/src/execution/
  - packages/database/src/schema/
  - packages/database/src/repositories/
  - packages/database/migrations/
  - apps/api/src/modules/runtime/
  - apps/workers/src/outbox.ts
  - tests/integration/
  - docs/contracts/
  - docs/backlog/
inputs:
  - Trusted actor, tenant, approved task type and normalized task payload
  - Tenant-scoped idempotency key and payload fingerprint
  - Accepted tenant and transaction-context contracts
outputs:
  - Durable run, input snapshot, job registry entry and outbox record
  - Stable job identity and safe status/result-reference contract
  - Scoped read, cancel and eligible recovery operations
invariants:
  - Commit the admitted run and outbox in one database transaction.
  - Do not create a public arbitrary-prompt or arbitrary-tool endpoint.
  - Duplicate delivery cannot duplicate domain artifacts.
  - Stale workers cannot finalize protected domain writes.
  - Redis loss does not erase the authoritative job registry.
acceptance_cases:
  - Identical request/key returns the original run.
  - Changed payload with the same key returns a conflict.
  - Failure after commit and before enqueue is recovered by the dispatcher.
  - Dispatcher crash after enqueue is safely redelivered.
  - Expired worker lease is recovered with a new fencing token.
  - Another tenant cannot inspect, cancel or recover this run.
  - Current revocation prevents a delayed consequential action.
  - Queue loss reconciles pending work from PostgreSQL.
deliverables:
  - File-level implementation plan and schema/API/job contracts
  - Reviewed migrations and implementation diff
  - Relevant verification commands and actual results
  - Recovery documentation and remaining limitations
out_of_scope:
  - Marketing generation logic
  - LangGraph graph implementation
  - Instagram publication
  - Production deployment or destructive data migration
```

Additional packages use the same structure and their own acceptance cases. Add `risk`, estimated size, rollout/rollback, and contract versions after the planner inspects the code.

### 21.3 First planner prompt

```text
Use this README as the proposed Synergy Blossom product architecture and the supplied v0 as the business requirements baseline.

First inspect the current repository, its applicable AGENTS.md instructions, installed ECC/harness configuration, and existing files. Do not assume the proposed directories, custom agents, slash commands, or scripts already exist.

Plan F0-01, F0-02, P0-01, P0-02, P0-03, and M0-01 through M0-03 in dependency order. The first working business slice is one brand and one month of text generation from approved brand/calendar fixtures. Preserve later platform boundaries without implementing every future module now.

Produce:
1. A repository/ECC capability inventory with verified vs proposed roles.
2. Architecture decisions needed for the first slice and unresolved business inputs.
3. Bounded implementation issues with IDs, files, ownership, dependencies, input/output contracts, edge cases, and acceptance checks.
4. A dependency graph and independent tasks eligible for parallel work.
5. A concrete first implementation task and its verification commands.

Keep Factory engineering agents separate from production business agents. Preserve existing ECC provenance/configuration and resolve file/command collisions deliberately. Do not claim unbuilt features or unrun tests are complete. Record assumptions rather than inventing client facts.

Return the plan for review. If the planner has read-only tools, the engineering harness/operator will save the plan and execute approved implementation tasks; do not assume the planner itself can write code or launch other agents.
```

### 21.4 Prompt for any later package

```text
Plan work package [PACKAGE_ID] from README section 19.

Read the applicable repository instructions, the package row, the linked requirements and relevant architecture/API/data/runtime sections. Inspect current implementation and accepted dependency evidence.

Keep its business outcome and architecture invariants. Identify exact files/functions, owned tables/migrations, runtime schemas/public interfaces, permissions, failure/retry/approval behavior, and relevant verification. Split into reviewable issues when needed. Assign one accountable owner per shared contract or migration.

List assumptions/blockers and any ADR required to change this baseline. Reuse existing platform services. Provide synthetic examples and acceptance cases. Do not implement unrelated future domains, bypass release policy, or create a second owner of execution/business data.

Output the implementation plan, ordered issue list, contract changes, testing/evaluation strategy, rollout/rollback notes, and definition of done. Distinguish proposed behavior from existing verified functionality.
```

### 21.5 Minimum planner output

The plan must contain the business trigger and resulting behavior, touched files, contract/schema versions, dependencies, ownership, permission model, invalid/missing input behavior, idempotency/version policy, retries/cancellation/uncertain effects where applicable, meaningful checks, expected evidence, and any unresolved decision that blocks dependent work.

Store accepted plans under `docs/backlog/<package-id>/plan.md`, contract details under `docs/contracts/`, and consequential choices under `docs/adr/`. A GitHub issue/PR carries the same package/requirement IDs. Stock read-only planners may return text for the harness to persist; this storage convention is a project procedure, not a claim that ECC automatically creates these files.

## 22. First build sequence and requirements traceability

### 22.1 Start here

| Order | Package(s) | Concrete action |
|---|---|---|
| 1 | F0-01, P0-02, M0-01 | Verify actual ECC/harness; establish pilot contracts; gather one approved brand/calendar dataset |
| 2 | F0-02, P0-01, P0-03 | Configure bounded Factory handoff; create compatible small TypeScript workspace; record core ADRs |
| 3 | M0-02 | Produce one monthly structured narrative and readable text with bounded usage |
| 4 | M0-03 | Inspect usefulness/facts with two reviewers; correct the contract before expansion |
| 5 | M1-01–M1-04 | Separate Time/Value and implement trustworthy references/validation/evaluations |
| 6 | M2 + P1 + F1 | Build local UI and shared identity/data foundations against stable contracts |
| 7 | P2 + M3 | Add durable monthly execution, revisions, review and authorized history |
| 8 | M4 + P3 | Deliver yearly outline/month expansion and the staff text release |
| 9 | P4 or P5/M5 according to demand | Establish external client readiness and/or advanced durable orchestration |
| 10 | M6–M9 and approved P6 workflows | Add researched inputs, creative/publication/measurement and new domains through their gates |

Use the architecture to protect future reuse, not to delay the first useful output. The first model call is a small controlled workflow; the first shared app is a governed product; the later agent orchestra grows from accepted contracts and observed needs.

### 22.2 Requirement map

| ID | Requirement | Specification | Initial delivering packages |
|---|---|---|---|
| SYS-01 | One internal/client enterprise platform with clear product/Factory separation | 1–3, 18 | F0-01, F0-02, P0-03 |
| SYS-02 | ECC, LangGraph, LangChain, OpenClaw, MCP responsibilities | 2, 9–10, 15, 18 | F1-01, P2-02, P5-01, P4-03 |
| SYS-03 | Complete compatible stack, monorepo, import ownership | 4–5, 16 | P0-01, P0-03, F1-02 |
| SYS-04 | Frontend forms, work/results/review, typed API | 11–14 | M2-01, M2-03, P1-05, M3-02, M3-04 |
| SYS-05 | Authentication, sessions, grants, tenant/delegated access | 7, 12 | P1-02, P1-03, P4-01 |
| SYS-06 | Owned schemas, versioning, RLS, transactional context | 8 | P1-01, P1-04, M3-01 |
| SYS-07 | Evidence/calendar/source provenance, ingestion and governed memory | 8, 13, 15 | P1-04, P4-02, M1-02, M1-03, M6-02 |
| SYS-08 | Durable runs, jobs/outbox/inbox, idempotency, crash recovery | 9, 12 | P2-01, M3-03, P3-04 |
| SYS-09 | Registered agents/tools, budgets, checkpoints, pause/resume | 9–10 | P2-02, P5-01, M5-01, M5-02 |
| SYS-10 | Exact-revision review and business authority | 9, 13–14 | P2-03, M3-04, M7-04 |
| SYS-11 | Whole-company demand intake, supported process routing | 6, 11–12 | P3-02, P5-01, P6-03 |
| SYS-12 | Audit, telemetry, fair limits, usage and cost accounting | 9, 17 | P2-01, P2-02, P2-04, P5-02, P4-04 |
| SYS-13 | Docker/hosted deployment, environment isolation, migration and restore | 16–17 | P3-01, P3-04, P4-04 |
| SYS-14 | Client onboarding, scoped APIs/connectors and data governance | 7–8, 12, 15, 17 | P4-01–P4-05 |
| MKT-01 | Brand/calendar-only monthly proof | 13 | M0-01–M0-03 |
| MKT-02 | Time and Value, explicit metrics and evidence boundaries | 13 | M1-01–M1-04 |
| MKT-03 | Persistent twelve-month outline plus selected-month text/review/export | 11, 13–14 | M3-01–M3-05, M4-01–M4-04 |
| MKT-04 | Durable/rolling planning, selective refresh and stale dependency handling | 9, 13–14 | M5-01–M5-03 |
| MKT-05 | Current reviewed trends, innovations and competitor context | 13–15 | M6-01–M6-03 |
| MKT-06 | Posts/carousels from exact reviewed text | 13–14 | M7-01–M7-04, P5-03 |
| MKT-07 | Official scheduled Instagram publication and reconciliation | 9, 13–15 | M0-04, M8-01–M8-04 |
| MKT-08 | Defined outcome metrics and reviewed improvements | 13, 17, 20 | M9-01–M9-03 |
| FAC-01 | Agent-driven planning, implementation, review and evidence | 18–22 | F0-01–F1-02 |
| FAC-02 | Sanitized engineering learning without production coupling | 18, 20 | F2-01 |
| DOM-01 | Additional domains through owned specifications and controlled pilots | 6, 12, 19–20 | P6-01–P6-04, P5-04 |

Every implementation issue references at least one requirement. Future domain routes, paid billing, OpenClaw hosts, semantic retrieval, video, dedicated client infrastructure, and arbitrary workflow builders require additional approved packages; they are not silently included in the first release.

## 23. Decisions, provenance, and technical references

### 23.1 Decisions still to close

The baseline choices above allow work to start. Resolve remaining decisions at their package gate; do not block unrelated work waiting for a later feature.

| Decision | Accountable owner | Due |
|---|---|---|
| Actual ECC fork commit, installed harness, roles/tools/paths/license and root-file collisions | Engineering owner | F0-01 |
| Pilot brand, audience, offer, market, language/local seasons, facts/public-use permission | Marketing owner | M0-01 |
| Exact model/provider configuration, token/cost limits and quality rubric | AI + product owner | M0; re-evaluate before P3 |
| Compatible package/container versions and ESM/test setup | Platform owner | P0-01 |
| Identity account provisioning, session/support/delegation policy and key ownership | Platform + business owner | P1; client review in P4 |
| Hosting region, plans, telemetry backend, data/trace retention, backup provision | Platform + business owner | P3; client review in P4 |
| Actual SLOs, run completion budgets, RPO/RTO and launch workload | Operations + QA | P3/P4 |
| Tenant-safe checkpoint implementation and workflow/state-version migration | Workflow + database owner | P5-01 |
| Research provider, accepted source classes and expiry/public-use rules | Strategist + integrations | M6 |
| Brand templates/dimensions/fonts and media rights | Designer + marketing owner | M7 |
| Current official Instagram login/account/scopes/review/media rules | Integrations + account owner | Feasibility M0-04; final gate M8 |
| Qualified leads, commercial data, attribution/denominators | Marketing analyst | M9 |
| Contracts/Finance/Operations process owners, data/actions and review authority | Respective domain owner | P6 discovery |
| Need for OpenClaw, semantic search, Python, video or commercial billing | Product + engineering owner | Only when a concrete demand justifies it |

These decisions become ADRs/configured contracts, not private assumptions in an agent prompt. Existing accepted decisions and user requirements take precedence over a Factory agent's preferred stack change.

### 23.2 Required ADRs before the shared release

Record modular API/worker boundaries; tenant/delegation and database role/RLS model; OIDC/session/proxy architecture; immutable revision/approval policy; initial provider/model and budget; outbox/retry/effect ownership; package/module compatibility; deployment topology/migration/rollback; data/log/trace retention. Before M5 add checkpoint isolation/version migration; before M8 add exact-package publication/reconciliation; before each new domain add its owned business-control model.

### 23.3 Reviewed primary references

Framework roles and selected infrastructure behavior were checked against primary documentation on **5 October 2026**. These references describe vendor/framework behavior; the Blossom-specific contracts, roles, architecture, work packages, and numerical targets are proposed engineering decisions in this document.

| Area | Primary reference | Relevance |
|---|---|---|
| ECC engineering system | [Upstream ECC](https://github.com/affaan-m/ECC) | Engineering assets and harness-oriented workflow; fork still needs local audit |
| ECC planning | [Planner definition](https://github.com/affaan-m/ECC/blob/main/agents/planner.md) | Requirement analysis, file-level steps, dependencies and acceptance planning |
| User fork | [LucasAlDev/ECC](https://github.com/LucasAlDev/ECC) | Intended Factory basis; exact current files could not be verified in public retrieval |
| Node.js | [Official release schedule](https://github.com/nodejs/Release/blob/main/README.md) | LTS runtime baseline |
| LangChain | [JavaScript overview](https://docs.langchain.com/oss/javascript/langchain/overview) | AI components and higher-level agents |
| LangGraph | [JavaScript overview](https://docs.langchain.com/oss/javascript/langgraph/overview) | Stateful orchestration; independent use possible |
| Graph state | [Persistence](https://docs.langchain.com/oss/javascript/langgraph/persistence) | Checkpointers versus stores; durable storage needed |
| Human waits | [Interrupts](https://docs.langchain.com/oss/javascript/langgraph/interrupts) | Pause/resume and node replay behavior |
| OpenClaw host | [Architecture](https://docs.openclaw.ai/concepts/architecture) | Channel/gateway operating model |
| OpenClaw isolation | [Security](https://docs.openclaw.ai/gateway/security) | One trusted boundary per gateway; unrelated clients require separation |
| MCP | [Architecture](https://modelcontextprotocol.io/docs/learn/architecture) | Host/client/server connectivity and primitives |
| PostgreSQL | [Row security policies](https://www.postgresql.org/docs/current/ddl-rowsecurity.html) | RLS, roles and bypass considerations |
| BullMQ | [Idempotent jobs](https://docs.bullmq.io/patterns/idempotent-jobs) | Safe repeated delivery and retry effects |
| Worker hosting | [Render background workers](https://render.com/docs/background-workers) | Continuously running worker deployment |
| Queue hosting | [Render Key Value](https://render.com/docs/key-value) | Managed compatible datastore and configuration checks |
| Frontend | [Next.js App Router](https://nextjs.org/docs/app) | Web routing/server rendering baseline |
| Backend | [NestJS documentation](https://docs.nestjs.com/) | Modular TypeScript API baseline |

Implementation reference points: [shadcn/ui](https://ui.shadcn.com/docs), [pnpm workspaces](https://pnpm.io/workspaces), [Drizzle](https://orm.drizzle.team/docs/overview), [Keycloak guides](https://www.keycloak.org/guides), [openid-client](https://github.com/panva/openid-client), [NestJS queues](https://docs.nestjs.com/techniques/queues), [NestJS OpenAPI](https://docs.nestjs.com/openapi/introduction), [OpenTelemetry](https://opentelemetry.io/docs/what-is-opentelemetry/), [transactional outbox](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html), and [official Meta Instagram developer documentation](https://developers.facebook.com/docs/instagram-platform/). Recheck exact APIs, package/license compatibility, provider permissions, pricing, and hosting limits when implementing each integration.

### 23.4 Completion boundary of this blueprint

This README gives the Factory a complete initial platform vision, reusable boundaries, the first application's requirements, and the work needed to start implementation. Additional domain-specific requirements are intentionally commissioned as discovery packages. The engineering Factory builds and verifies software; the Blossom platform owns and runs governed business processes; accountable business owners decide which outputs and actions are authorized.
