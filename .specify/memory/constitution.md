<!--
Sync Impact Report
- Version change: 0.0.0 → 1.0.0 (initial ratification)
- Added principles: Code Quality, Testing Standards, UX Consistency, Performance
- Added sections: Architecture Constraints, Development Workflow
- Templates requiring updates:
  - .specify/templates/plan-template.md: ✅ no updates needed (Constitution Check gate already present)
  - .specify/templates/spec-template.md: ✅ no updates needed (success criteria align with principles)
  - .specify/templates/tasks-template.md: ✅ no updates needed (phase structure compatible)
  - .specify/templates/checklist-template.md: ✅ no updates needed (generic structure)
  - .specify/templates/agent-file-template.md: ✅ no updates needed
- Follow-up TODOs: none
-->

# Quarkus Super Heroes Constitution

## Core Principles

### I. Code Quality

All production code MUST follow existing patterns and conventions established
in the codebase. Specifically:

- CDI beans MUST use constructor injection. Field injection is prohibited
  except for `@ConfigProperty`, `@Channel`, and test-only annotations.
- REST clients MUST use the MicroProfile REST Client with Stork service
  discovery. gRPC clients MUST use Quarkus gRPC with `@GrpcClient`.
- Fault tolerance (Circuit Breaker, Retry, Timeout, Fallback) MUST be
  applied to all external service calls. Fallback values MUST be
  configurable via `application.properties`.
- MapStruct MUST be used for mapping between domain entities and schema
  objects. Manual mapping code is prohibited where MapStruct applies.
- Code changes MUST NOT introduce compiler warnings. Deprecation warnings
  from upstream dependencies are exempt.

### II. Testing Standards

Every module MUST maintain a minimum of 80% code coverage. Test structure
MUST follow these rules:

- Unit tests (`*Tests.java`) MUST use `@QuarkusTest` with mocked
  dependencies (WireMock for HTTP, WireMock gRPC for gRPC, in-memory
  channels for Kafka).
- Integration tests (`*IT.java`) MUST use `@QuarkusIntegrationTest` with
  real infrastructure via Testcontainers (Kafka, MongoDB).
- Tests MUST be locale-independent. Surefire `argLine` MUST include
  `-Duser.language=en -Duser.country=US` in modules with Bean Validation
  message assertions.
- Tests MUST NOT depend on execution order. Each test MUST reset shared
  state (e.g., WireMock stubs) in `@BeforeEach`.
- Builds MUST always use `mvn clean` to avoid MapStruct incremental
  compilation corruption.

### III. UX Consistency

All microservice APIs MUST present a uniform interface to consumers:

- REST endpoints MUST expose OpenAPI documentation via SmallRye OpenAPI
  with title, description, version, and contact metadata.
- All REST services MUST include `/api/{resource}/hello` health-check
  endpoints returning a greeting string.
- Error responses MUST use standard HTTP status codes: 404 for not found,
  400 for validation failures, 500 for unexpected errors.
- CORS MUST be enabled on all REST services.
- The UI MUST display real-time fight results. WebSocket endpoints MUST
  broadcast statistics updates to all connected clients.
- JSON serialization MUST use `non-empty` inclusion to omit null/empty
  fields from responses.

### IV. Performance

Services MUST meet latency and resilience targets under normal operation:

- External service calls MUST have explicit timeouts (max 5s for hello
  endpoints, max 2s for data fetches, max 30s for narration/image
  generation).
- Circuit breakers MUST open after 50% failure ratio over 8 requests,
  with a 2-second delay before half-open.
- Retry policies MUST NOT exceed 3 retries with 200ms delay between
  attempts.
- Kafka message publishing MUST be non-blocking. The fight service MUST
  use `sendAndForget` for event emission.
- MongoDB queries MUST use reactive Panache (`ReactivePanacheMongoEntity`)
  for non-blocking database access.
- OpenTelemetry tracing MUST be enabled on all service methods annotated
  with `@WithSpan` to support performance monitoring.

## Architecture Constraints

The project follows a microservices architecture with these boundaries:

- **rest-fights**: Orchestrator service. Calls rest-heroes, rest-villains
  (via REST + Stork), rest-narration (via REST + Stork), and
  grpc-locations (via gRPC). Publishes fight events to Kafka.
- **rest-heroes**, **rest-villains**: CRUD services with PostgreSQL
  backends via Hibernate Reactive Panache.
- **rest-narration**: AI-powered narration service (pluggable backends).
- **grpc-locations**: gRPC service providing random fight locations.
- **event-statistics**: Kafka consumer computing real-time fight
  statistics, exposed via WebSocket.

Cross-cutting concerns:
- Service discovery MUST use SmallRye Stork with static list (prod),
  Microcks (dev), or WireMock (test).
- Schema evolution for Kafka messages MUST use Avro with Apicurio Registry.
- Each module MUST be independently buildable and testable.

## Development Workflow

- All changes MUST compile and pass tests before committing.
- The canonical build command is:
  `mvn clean install -DskipTests && mvn test --projects rest-fights,rest-heroes,rest-villains,rest-narration,event-statistics,grpc-locations`
  (two-step build due to Quarkus 3.31 FacadeClassLoader reactor bug).
- Docker MUST be running for integration tests that use Testcontainers.
- Commits MUST be atomic, focused on a single change, with messages
  explaining why (not what).
- Feature branches MUST branch from `main`. Force-pushing to `main` is
  prohibited.

## Governance

This constitution is the authoritative reference for all development
decisions on the Quarkus Super Heroes project. When in doubt, principles
in this document take precedence over ad-hoc decisions.

- Amendments MUST be documented with a version bump, rationale, and
  updated Sync Impact Report.
- All pull requests MUST verify compliance with these principles.
- Complexity beyond what these principles prescribe MUST be justified
  in the PR description.
- The `.specify/memory/constitution.md` file is the single source of
  truth. Runtime guidance is maintained separately in CLAUDE.md.

**Version**: 1.0.0 | **Ratified**: 2026-02-07 | **Last Amended**: 2026-02-07
