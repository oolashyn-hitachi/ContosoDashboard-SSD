# Implementation Plan: Document Upload and Management

**Branch**: `001-document-management` | **Date**: 2026-10-05 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/001-document-management/spec.md`

## Summary

Add an offline, training-only document library to ContosoDashboard. Employees upload, organize,
search, preview, download, edit, replace, delete, and share work files. Project/task context,
department-based sharing, in-app notifications, dashboard summaries, and administrator activity
reports reuse the existing Blazor Server application and authorization/data layers. Document bytes
remain outside the public web root behind local storage and per-document authorization. Upload
screening is a clearly labeled simulation and does not detect malware.

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: C# on .NET 8

**Primary Dependencies**: ASP.NET Core 8 Blazor Server; EF Core 8 SQL Server; xUnit; EF Core SQLite

**Storage**: SQL Server LocalDB for metadata; local filesystem outside `wwwroot` for document bytes

**Testing**: New .NET 8 xUnit project with SQLite in-memory data tests and temporary-directory
storage tests; manual Blazor end-to-end validation in `quickstart.md`

**Target Platform**: Local, offline training environment using the existing Windows/LocalDB setup

**Project Type**: Single-project Blazor Server web application plus a focused test project

**Performance Goals**: 25 MiB upload in 30 seconds; lists of up to 500 documents and searches within
2 seconds; PDF/image previews within 3 seconds, as measured in the feature spec

**Constraints**: No cloud or external service dependency; simulated screening MUST NOT claim real
malware detection; per-document authorization applies to metadata and file content; existing
LocalDB databases require an explicit backup/reset to acquire the new schema

**Scale/Scope**: Four user stories; documents, department shares, tags, and activity records;
per-user lists up to 500 documents

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **Training Scope and Honest Limitations**: PASS. The feature is local/offline and marks screening
  as a simulation; it makes no production-readiness or malware-detection claim.
- **Security and Data Isolation**: PASS. Document authorization is checked in the service boundary
  and content endpoints. Planned tests cover owner, project, department, administrator, denied, and
  stale-membership access.
- **Layered, Minimal Architecture**: PASS. The feature extends the existing Models, Data, Services,
  Pages, and Shared layers. Storage and screening abstractions exist only at genuine boundaries.
- **Verifiable Behavior**: PASS. A small automated test project is justified because there is no
  existing harness and authorization, persistence, and file access are security-sensitive. Manual
  UI checks cover complete Blazor workflows.
- **Offline-First Dependencies**: PASS. Runtime dependencies remain local; no Azure, cloud, or
  external scanning service is introduced.
- **Pre-research gate**: PASS. No constitution deviation is required.
- **Post-design gate**: PASS. The entity model, protected content contract, offline quickstart, and
  test strategy preserve all five principles; no new deviation was introduced during design.

## Project Structure

### Documentation (this feature)

```text
specs/001-document-management/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/           # Phase 1 output (/speckit-plan command)
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
ContosoDashboard/
├── Data/ApplicationDbContext.cs
├── Models/
│   ├── Document.cs
│   ├── DocumentTag.cs
│   ├── DocumentShare.cs
│   └── DocumentActivity.cs
├── Services/
│   ├── DocumentService.cs
│   ├── IFileStorageService.cs
│   ├── LocalFileStorageService.cs
│   └── TrainingUploadScreeningService.cs
├── Endpoints/DocumentContentEndpoints.cs
├── Pages/
│   ├── Documents.razor
│   ├── Index.razor
│   ├── ProjectDetails.razor
│   └── Tasks.razor
├── Shared/NavMenu.razor
└── Program.cs

ContosoDashboard.Tests/
├── ContosoDashboard.Tests.csproj
├── Services/DocumentServiceTests.cs
└── Services/LocalFileStorageServiceTests.cs

specs/001-document-management/
├── contracts/document-management.md
├── data-model.md
├── quickstart.md
└── research.md
```

**Structure Decision**: Extend the existing Blazor Server project in place. A single
`Documents.razor` page provides My Documents and Shared with Me views; existing project and task
pages host context-specific document panels; the dashboard adds recent documents and a count. A
`DocumentContentEndpoints` module streams authorized content without exposing storage paths.
`DocumentService` owns metadata queries, access policy, activity, and notifications; the storage and
screening services own only their respective boundaries. The test project is justified by the
constitution's verification gate and the absence of an existing harness.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | No constitution violations | N/A |
