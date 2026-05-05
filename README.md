# Sanatry – System Design Overview

## 1. System Purpose and Scope
Sanatry is a private, multi-tenant SaaS backend for education center operations. It serves multiple organizations within a single deployment while preserving strict data boundaries. The design prioritizes correctness of tenant isolation and predictable behavior under concurrent use.

## 2. Architecture & Component Interaction
- Requests enter through a thin HTTP layer that authenticates, validates, and normalizes input before invoking a service boundary.
- Services coordinate multi-step workflows, enforce invariants, and define transactional scope; they are the primary unit of business change.
- Data access is centralized so tenant scoping, soft deletion, and validation are consistently applied across read and write paths.
- Responses are shaped at the boundary layer, keeping transport concerns separate from domain decisions.

## 3. Multi-Tenancy Strategy
- Tenant identity is derived from the authenticated principal and carried through the request context.
- Isolation is enforced in the data-access layer, ensuring every query is scoped and every write is validated against tenant ownership.
- Risks include cross-tenant leakage and implicit joins; mitigations are centralized scoping, explicit ownership checks, and targeted tests for boundary conditions.

## 4. Core Design Decisions
- Service layer: chosen to keep views thin and to prevent business logic from fragmenting across endpoints, which improves auditability and testability.
- Data modeling: favors explicit relationships and stable identifiers to support auditing, soft deletion, and long-lived references.
- Structured response contracts: chosen to make error handling predictable for clients and to keep failure semantics consistent across modules.
- Trade-off: a stronger service boundary adds more internal orchestration, but reduces duplication and accidental divergence in behavior.

## 5. Concurrency & Consistency Model
- Critical workflows execute within database transactions to preserve invariants across multi-step updates.
- Explicit locking is used where concurrent writes could violate capacity or scheduling constraints.
- Non-critical reads remain optimistic to avoid unnecessary contention; only state transitions that must be serialized pay the locking cost.
- Trade-off: stricter consistency for sensitive flows increases contention under peak load, so it is applied selectively.

## 6. Security Model
- Authentication is token-based with browser-safe cookies to reduce exposure of credentials to client-side scripts.
- CSRF protection is paired with cookie sessions to protect state-changing operations in web contexts.
- Step-up verification and device trust are used to balance friction with risk, while keeping verification paths revocable and auditable.

## 7. CI/CD & Delivery Pipeline
- Each commit triggers automated checks; changes are exercised in a containerized environment to keep runtime conditions consistent.
- GitHub Actions orchestrates the pipeline, with Docker providing parity between development and test execution.
- This flow reduces environment drift and makes regressions observable before code merges.

## 8. Testing Strategy
- Tests treat the service layer as the primary unit of correctness, validating invariants and state transitions directly.
- Integration tests focus on authentication, tenant boundaries, and workflows that combine multiple services.
- Coverage targets around 85% with priority on security-sensitive and concurrency-prone paths.
- Boundary value analysis and equivalence partitioning guide test case selection to maximize behavioral coverage with limited test volume.
