<!--
Sync Impact Report
Version change: unversioned template -> 1.0.0 (first complete project constitution)
Modified principles: none (the previous document contained only template placeholders)
Added principles: Training Scope and Honest Limitations; Security and Data Isolation;
	Layered, Minimal Architecture; Verifiable Behavior; Offline-First Dependencies
Added sections: Training and Security Constraints; Development Workflow and Quality Gates
Removed sections: none
Template/runtime review:
	.specify/templates/plan-template.md - reviewed; no update required, gate is constitution-driven
	.specify/templates/spec-template.md - reviewed; no update required
	.specify/templates/tasks-template.md - reviewed; no update required
	.specify/templates/commands/ - directory absent; no command files to review
	README.md - reviewed; no update required
Deferred: TODO(RATIFICATION_DATE): original adoption date is unknown
-->
# ContosoDashboard Constitution

## Core Principles

### I. Training Scope and Honest Limitations
ContosoDashboard MUST remain clearly scoped as a fictional training application. Features and
documentation MUST identify mock or simplified behavior and MUST NOT imply production readiness.
This keeps learners from mistaking educational shortcuts for production controls.

### II. Security and Data Isolation
Pages that expose protected data MUST require authorization, and services MUST enforce authorization
for protected operations. Changes to access control MUST preserve user and project data isolation and
MUST include verification of allowed and denied access. Mock authentication MUST remain identified as
training-only and MUST NOT be presented as a production identity system.

### III. Layered, Minimal Architecture
Changes MUST preserve clear responsibilities across the existing Pages, Services, Data, and Models
layers. Infrastructure that needs to vary between local and cloud environments MUST be accessed
through an appropriate abstraction. New abstractions and dependencies MUST solve a demonstrated
requirement; avoid speculative complexity in this training codebase.

### IV. Verifiable Behavior
Every feature plan MUST define repeatable verification for its acceptance criteria. Changes to
authorization, data isolation, persistence, or business rules MUST include automated tests when a
test harness is available; otherwise the plan MUST document executable manual checks. Verification
MUST cover relevant failure and denied-access paths, not only successful use.

### V. Offline-First Dependencies
The default application MUST remain usable for local training without cloud accounts or external
service dependencies. A feature that introduces an external dependency MUST justify it in its spec
and preserve a practical local development and verification path.

## Training and Security Constraints

The application uses simplified authentication and local infrastructure for training. New features
MUST NOT add real credentials, weaken existing authorization boundaries, or claim compliance or
production security guarantees that the application does not provide. Any known security limitation
introduced or changed by a feature MUST be stated in its documentation and acceptance criteria.

## Development Workflow and Quality Gates

Feature specifications MUST state observable requirements and acceptance criteria. Plans MUST
include a Constitution Check and record any justified deviation. Tasks MUST map implementation and
verification to the planned user stories. Before completion, contributors MUST run the applicable
build and verification checks and document any checks that could not be run. Reviews MUST confirm
that changes comply with this constitution and that training-only limitations remain clear.

## Governance

This constitution governs feature specifications, plans, implementation, and review. Amendments
require project-owner approval and MUST document the rationale and impact in the Sync Impact Report.
The project owner MUST review proposed changes for compatibility with existing features and update
dependent guidance only when the amendment changes its required structure or rules. Reviewers MUST
check compliance and require a documented rationale for deviations.

The constitution follows semantic versioning: MAJOR for incompatible principle or governance
changes, MINOR for added principles or materially expanded requirements, and PATCH for clarifications
that do not change obligations. Every amendment MUST update the version and last-amended date.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): original adoption date unknown | **Last Amended**: 2026-10-05
