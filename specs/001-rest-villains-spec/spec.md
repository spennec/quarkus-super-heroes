# Feature Specification: REST Villains Microservice

**Feature Branch**: `001-rest-villains-spec`
**Created**: 2026-02-08
**Status**: Draft
**Input**: User description: "REST Villains Microservice - A standalone CRUD microservice for managing super-villains in the Super Heroes application"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Retrieve a Random Villain for Combat (Priority: P1)

The fight orchestrator service requests a randomly-selected villain to participate in a fight. The system selects one villain at random from the entire villain roster and returns its full profile. If no villains exist in the system, the service communicates that no villain is available.

**Why this priority**: This is the core integration point with the fight system. Without random villain selection, fights cannot be generated — the primary purpose of the villain service within the Super Heroes ecosystem.

**Independent Test**: Can be fully tested by populating the villain roster and requesting a random villain multiple times, verifying each response is a valid villain with all required fields. Delivers the essential value of providing opponents for fights.

**Acceptance Scenarios**:

1. **Given** one or more villains exist in the system, **When** a random villain is requested, **Then** the system returns a single complete villain profile with all fields populated (id, name, level at minimum).
2. **Given** multiple villains exist in the system, **When** random villain is requested many times, **Then** different villains are returned over time (not always the same one).
3. **Given** no villains exist in the system, **When** a random villain is requested, **Then** the system responds with a "not found" indication.

---

### User Story 2 - Manage Individual Villains (Priority: P1)

An administrator creates, views, updates, and deletes individual villains in the roster. When creating a villain, the system applies a configurable difficulty adjustment to the villain's power level. Updates can be either full replacements or partial modifications that only change specified fields.

**Why this priority**: CRUD operations are fundamental to maintaining the villain roster. Without the ability to manage villains, the system has no data to serve.

**Independent Test**: Can be fully tested by creating a villain, retrieving it, updating it (both fully and partially), and deleting it — verifying correct behavior at each step. Delivers the value of a self-sustaining villain management system.

**Acceptance Scenarios**:

1. **Given** valid villain data (name between 3-50 characters, positive power level), **When** a new villain is created, **Then** the system stores the villain with an auto-generated unique identifier, applies the level multiplier to the power level, and returns a reference to the created resource.
2. **Given** a villain exists with id X, **When** the villain is retrieved by id X, **Then** the system returns the complete villain profile.
3. **Given** a villain exists with id X, **When** a full update is submitted with valid data, **Then** all fields of the villain are replaced with the new values (except the identifier).
4. **Given** a villain exists with id X and has name "Darth Vader" and level 5, **When** a partial update is submitted with only name "Darth Sidious", **Then** the name changes to "Darth Sidious" but the level remains 5 and all other fields are preserved.
5. **Given** a villain exists with id X, **When** the villain is deleted, **Then** the villain is removed and subsequent retrieval by id X returns "not found."
6. **Given** no villain exists with id X, **When** a retrieval, update, or delete is attempted for id X, **Then** the system responds with "not found."
7. **Given** villain data with a name shorter than 3 characters or missing a power level, **When** creation or update is attempted, **Then** the system rejects the request with a validation error.

---

### User Story 3 - Browse and Filter Villain Roster (Priority: P2)

A user browses the full roster of villains or narrows the list by searching for villains whose names match a given filter. The name search is case-insensitive so that users do not need to know exact capitalization.

**Why this priority**: Browsing and filtering enable discovery and management of the villain roster, but the system can function without them (individual retrieval and random selection cover the primary use cases).

**Independent Test**: Can be fully tested by populating the roster, listing all villains, and filtering by various name fragments — verifying the correct subset is returned. Delivers the value of discoverability and search.

**Acceptance Scenarios**:

1. **Given** multiple villains exist, **When** the full roster is requested without a filter, **Then** all villains are returned.
2. **Given** villains named "Darth Vader", "Dark Phoenix", and "Thanos" exist, **When** the roster is filtered by "dark", **Then** both "Darth Vader" and "Dark Phoenix" are returned (case-insensitive match).
3. **Given** villains exist but none match the filter "zzzzz", **When** the roster is filtered by "zzzzz", **Then** an empty list is returned.
4. **Given** no villains exist, **When** the full roster is requested, **Then** an empty list is returned.

---

### User Story 4 - Bulk Villain Operations (Priority: P3)

An administrator replaces the entire villain roster with a new set of villains or clears the roster entirely. This supports data migration, reset scenarios, and test data management.

**Why this priority**: Bulk operations are administrative utilities. They are important for data lifecycle management but are not required for day-to-day operation.

**Independent Test**: Can be fully tested by populating the roster, performing a bulk replace with a new set, verifying the old data is gone and new data is present, then deleting all and confirming an empty roster. Delivers the value of efficient bulk data management.

**Acceptance Scenarios**:

1. **Given** villains exist in the system, **When** a bulk replace is submitted with a new list of valid villains, **Then** all existing villains are removed and the new villains are persisted, returning a reference to the new collection.
2. **Given** villains exist in the system, **When** a delete-all operation is performed, **Then** all villains are removed and the roster is empty.
3. **Given** a bulk replace is submitted with invalid villain data in the list, **When** validation fails, **Then** the entire operation is rejected and no changes are made.

---

### User Story 5 - Service Health Monitoring (Priority: P2)

Operations teams monitor the health of the villain service by calling a dedicated health-check endpoint. The endpoint returns a simple greeting that confirms the service is alive and able to handle requests.

**Why this priority**: Health checking enables automated monitoring, load balancing, and container orchestration. It is critical for production operations but does not affect end-user functionality.

**Independent Test**: Can be fully tested by calling the health endpoint and verifying a greeting response is returned. Delivers the value of operational visibility.

**Acceptance Scenarios**:

1. **Given** the villain service is running, **When** the health endpoint is called, **Then** a text greeting is returned confirming the service is operational.

---

### Edge Cases

- What happens when the level multiplier reduces a villain's level to zero? The system rounds using standard rules. If the result is 0, it is stored as 0 — the positivity constraint applies to the input value, not the stored value after transformation.
- What happens when a partial update submits an empty body (no fields)? The villain remains unchanged and the response returns the unmodified villain.
- What happens when a partial update changes the name to fewer than 3 characters? The system validates the *resulting* entity after merging, rejecting it with a validation error even though the original entity was valid.
- How does the system behave under concurrent modifications to the same villain? Standard database-level transactional isolation handles concurrency; no application-level locking is specified.
- What happens when a bulk replace is submitted with an empty list? All existing villains are deleted and the roster becomes empty (equivalent to delete-all).
- What happens when the name filter is an empty string? Treated as "no filter" — all villains are returned.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST provide a way to retrieve a single randomly-selected villain from the roster.
- **FR-002**: The system MUST return a "not found" response when a random villain is requested but no villains exist.
- **FR-003**: The system MUST allow creation of a new villain with the fields: name (required, 3-50 characters), level (required, positive integer), otherName (optional), picture (optional), and powers (optional).
- **FR-004**: The system MUST auto-generate a unique identifier for each newly created villain.
- **FR-005**: The system MUST apply a configurable level multiplier to the villain's level upon creation, rounding to the nearest integer. The default multiplier MUST be 0.5.
- **FR-006**: The system MUST NOT apply the level multiplier during update operations (full or partial).
- **FR-007**: The system MUST allow retrieval of a single villain by its unique identifier.
- **FR-008**: The system MUST allow full replacement of a villain's data (all fields except identifier) via a full update operation.
- **FR-009**: The system MUST allow partial updates where only the provided (non-null) fields are changed, preserving all other existing field values.
- **FR-010**: The system MUST validate the resulting entity after a partial update merge and reject it if constraints are violated.
- **FR-011**: The system MUST allow deletion of a single villain by its unique identifier.
- **FR-012**: The system MUST allow listing all villains in the roster.
- **FR-013**: The system MUST support optional case-insensitive name filtering when listing villains, matching any villain whose name contains the filter text.
- **FR-014**: The system MUST allow bulk replacement of the entire roster with a new set of villains in a single operation.
- **FR-015**: The system MUST allow deletion of all villains in a single operation.
- **FR-016**: The system MUST return a "not found" response when attempting to retrieve, update, or delete a villain that does not exist.
- **FR-017**: The system MUST reject creation or update requests that violate data constraints (name length, level positivity) with a validation error response.
- **FR-018**: The system MUST provide a health-check endpoint that returns a text greeting confirming the service is operational.
- **FR-019**: The system MUST omit null or empty fields from response payloads.
- **FR-020**: The system MUST expose machine-readable API documentation describing all endpoints, operations, and data schemas.
- **FR-021**: The system MUST accept cross-origin requests from any domain.
- **FR-022**: The system MUST return a reference (location) to newly created resources upon successful creation.
- **FR-023**: The system MUST support distributed tracing across all service operations for observability.
- **FR-024**: The system MUST expose operational metrics for monitoring (request rates, latencies, error rates).
- **FR-025**: The system MUST include trace context (trace ID, span ID) in structured log output for log correlation.

### Key Entities

- **Villain**: Represents a super-villain in the roster. Key attributes:
  - **Identifier**: System-generated unique ID, immutable once assigned
  - **Name**: The villain's primary name (required, 3-50 characters)
  - **Other Name**: An alternate name or alias (optional, no length constraint beyond storage limits)
  - **Level**: Numeric power level indicating the villain's strength (required, must be a positive integer on input; stored value may differ after multiplier application)
  - **Picture**: A reference (URL) to the villain's image (optional)
  - **Powers**: A textual description of the villain's abilities (optional, free-form text, supports large values)

  **Relationships**: Villain is a standalone entity with no relationships to other entities within this service. It is consumed externally by the fight orchestrator service.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A randomly-selected villain is returned within 500 milliseconds under normal load, 95% of the time.
- **SC-002**: All CRUD operations (create, read, update, delete) complete within 300 milliseconds under normal load, 95% of the time.
- **SC-003**: The system correctly validates all data constraints, rejecting 100% of invalid inputs with appropriate error responses.
- **SC-004**: The name filter returns accurate results — when filtering by a substring, 100% of villains whose names contain that substring (case-insensitive) are returned and no false positives are included.
- **SC-005**: The level multiplier is applied exactly once during creation and never during updates, verifiable by creating a villain and updating it without the level changing unexpectedly.
- **SC-006**: Partial updates preserve 100% of unspecified fields — no data is lost when only a subset of fields is provided.
- **SC-007**: The health-check endpoint responds within 100 milliseconds, 99% of the time, confirming operational readiness.
- **SC-008**: The system can serve the full villain roster (100+ entries) in a single response within 1 second.
- **SC-009**: All service operations are traceable end-to-end, with trace context appearing in both outbound responses and log entries.
- **SC-010**: The API documentation is auto-generated, always current, and accessible without authentication.
- **SC-011**: The system starts up and becomes ready to serve requests within 30 seconds (JVM mode) or 1 second (native mode).
- **SC-012**: Consumer contract tests pass, ensuring compatibility with the fight orchestrator service's expectations.

## Assumptions

- The villain roster is expected to contain hundreds of entries, not millions. Pagination is not required for the list endpoint.
- The level multiplier is a deployment-time configuration value, not a runtime-adjustable setting.
- No authentication or authorization is required to access any endpoint. The service operates in a trusted internal network.
- The "random" selection does not need to be cryptographically secure — pseudo-random is acceptable.
- Pre-loaded sample data (approximately 100 villains) is provided for initial deployment and development/testing purposes.
- The service is stateless from a session perspective; all state is persisted in the database.
- Concurrent access is handled by the underlying database (standard transactional isolation); no application-level locking is specified.
- The identifier is numeric (integer/long), auto-incremented via a sequence mechanism.
- The bulk replace operation is atomic — either all new villains are persisted or the operation fails entirely.

## Dependencies

- **Downstream Consumer**: The fight orchestrator service consumes the random villain and hello endpoints. Contract compatibility must be maintained.
- **Database**: A relational database is required for persistent storage.
- **Observability Infrastructure**: A tracing collector and metrics scraper are expected in the deployment environment (degradation is graceful if unavailable).

## API Contract Summary

The service exposes a REST API with the following operations:

| Operation              | Method | Path                        | Request Body    | Success Response                        | Error Responses                          |
|------------------------|--------|-----------------------------|-----------------|-----------------------------------------|------------------------------------------|
| List all villains      | GET    | /api/villains               | None            | 200 - Array of villains                 | None                                     |
| Filter villains        | GET    | /api/villains?name_filter=X | None            | 200 - Filtered array of villains        | None                                     |
| Get villain by ID      | GET    | /api/villains/{id}          | None            | 200 - Single villain                    | 404 - Not found                          |
| Get random villain     | GET    | /api/villains/random        | None            | 200 - Single villain                    | 404 - No villains exist                  |
| Create villain         | POST   | /api/villains               | Villain (no id) | 201 - Created villain + Location header | 400 - Validation error                   |
| Full update villain    | PUT    | /api/villains/{id}          | Villain (no id) | 204 - No content                        | 400 - Validation error, 404 - Not found  |
| Partial update villain | PATCH  | /api/villains/{id}          | Partial villain | 200 - Updated villain                   | 400 - Validation error, 404 - Not found  |
| Delete villain         | DELETE | /api/villains/{id}          | None            | 204 - No content                        | None                                     |
| Replace all villains   | PUT    | /api/villains               | Array of villains | 201 - Created + Location header       | 400 - Validation error                   |
| Delete all villains    | DELETE | /api/villains               | None            | 204 - No content                        | None                                     |
| Health check           | GET    | /api/villains/hello         | None            | 200 - Text greeting                     | None                                     |

**Villain Schema** (for request/response bodies):

| Field     | Type    | Required on Create | Constraints                | Notes                                     |
|-----------|---------|--------------------|----------------------------|-------------------------------------------|
| id        | Integer | No (auto-generated)| System-assigned, immutable | Present in responses only                 |
| name      | String  | Yes                | 3-50 characters, non-null  | Primary display name                      |
| otherName | String  | No                 | No length constraint       | Alias, omitted from response if empty     |
| level     | Integer | Yes                | Positive (> 0) on input   | Multiplied by configurable factor on create |
| picture   | String  | No                 | No constraint              | URL to image, omitted if empty            |
| powers    | String  | No                 | No constraint              | Free-form text, omitted if empty          |

## Database Requirements

- **Storage**: A single table storing villain records with columns matching the entity fields.
- **Primary Key**: Auto-generated numeric identifier using a sequence (start: 1, increment: 50).
- **Constraints**: Name column is NOT NULL with max length 50. Level column is NOT NULL. Other columns are nullable.
- **Powers field**: Must support large text values (not limited to 255 characters).
- **Initial Data**: The database must be seeded with approximately 100 sample villain records for development and demonstration purposes.
- **Schema Lifecycle**: In development/test environments, the schema is dropped and recreated on each startup. In production, the schema is validated against the entity model without modification.

## Configuration Requirements

- **Level Multiplier**: A configurable decimal value applied to villain levels on creation. Default: 0.5. Must be overridable per environment.
- **Service Port**: Configurable HTTP port. Default: 8084.
- **Cross-Origin Requests**: CORS enabled for all origins by default.
- **JSON Serialization**: Null and empty fields omitted from responses.
- **Tracing**: Configurable tracing collector endpoint. Tracing must include database operations.
- **Logging**: Debug-level logging for service operations. Production logging must include trace correlation context.
- **API Documentation**: Interactive API explorer always available (not just in development mode).

## Non-Functional Requirements

- **Deployment Flexibility**: The service must be deployable as a standalone container image in both JVM and native compilation modes. It must support orchestration platforms including standard container orchestration, enterprise PaaS, and serverless (scale-to-zero) environments.
- **Startup Performance**: Native compilation mode should achieve sub-second startup. JVM mode should start within 30 seconds.
- **Operational Observability**: The service must expose health check, metrics, and tracing endpoints out of the box without additional configuration.
- **Consumer Compatibility**: Contract tests must verify compatibility with known consumer expectations (fight orchestrator). Contract test definitions are maintained as versioned artifacts.
- **Data Isolation**: The service owns its database exclusively. No other service reads from or writes to the villain database.
- **Graceful Degradation**: If the tracing collector or metrics scraper is unavailable, the service must continue to function normally without errors.
