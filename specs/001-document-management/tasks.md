# Tasks: Document Upload and Management

**Input**: Design documents from `specs/001-document-management/`

**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/document-management.md`, `quickstart.md`

**Tests**: Automated tests are included because the approved plan introduces an xUnit/SQLite test project and the constitution requires automated coverage for authorization, isolation, and persistence when a harness is available. Each story's tests precede its implementation tasks.

**Organization**: Tasks are grouped by the four prioritized user stories in `spec.md`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Independent work in separate files with no unfinished task dependency.
- **[Story]**: Story traceability label; setup, foundational, and polish tasks omit it.
- Every task names its target file path.

## Phase 1: Setup

**Purpose**: Create the test harness and local storage configuration required by all stories.

- [ ] T001 [P] Create `ContosoDashboard.Tests/ContosoDashboard.Tests.csproj` targeting `net8.0`, referencing `ContosoDashboard/ContosoDashboard.csproj`, and adding xUnit, the .NET test SDK, and `Microsoft.EntityFrameworkCore.Sqlite` packages.
- [ ] T002 [P] Add a configurable local upload root outside `wwwroot` to `ContosoDashboard/appsettings.json`, defaulting to `AppData/uploads` beneath the application content root.

**Checkpoint**: The app and empty test project restore independently; the configured file root is not web-accessible.

---

## Phase 2: Foundational

**Purpose**: Add shared entities, local storage and screening boundaries, EF configuration, and dependency injection. All user stories depend on this phase.

- [ ] T003 [P] Add `Document` in `ContosoDashboard/Models/Document.cs` with data-model constraints verbatim: `DocumentId: Integer, primary key`; `Title: String, required, max 255`; `Description: Nullable string, max 2000`; `Category: String, required` with values Project Documents, Team Resources, Personal Files, Reports, Presentations, Other; `OriginalFileName: String, max 255`; `RelativeFilePath: String, required, max 512, unique` using `{userId}/{projectId or "personal"}/{guid}.{ext}`; `FileSizeBytes: Long, required`, greater than zero and at most 26,214,400 bytes; `FileType: String, required, max 255`; `UploadedAtUtc: UTC date/time, required`; `UploadedByUserId: Integer, required FK to User`; `ProjectId: Nullable integer FK to Project`; and `TaskId: Nullable integer FK to Task`, with a task's project required to match the document's project.
- [ ] T004 [P] Add `DocumentTag` in `ContosoDashboard/Models/DocumentTag.cs` with `DocumentTagId: Integer, primary key`; `DocumentId: Integer, required FK to Document`; and `Value: String, required, max 100`, trimmed and unique per document after normalization.
- [ ] T005 [P] Add `DocumentActivity` in `ContosoDashboard/Models/DocumentActivity.cs` with `DocumentActivityId: Integer, primary key`; `DocumentId: Nullable integer, no cascading FK`; `DocumentTitleSnapshot: String, max 255`; `ActorUserId: Integer, required FK to User`; `Action: String or enum, required`; `OccurredAtUtc: UTC date/time, required`; and `Details: Nullable string, max 1000` containing no file content or local path.
- [ ] T006 [P] Define `IFileStorageService` in `ContosoDashboard/Services/IFileStorageService.cs` with upload, delete, download, and protected-route URL operations that accept/return relative storage keys only.
- [ ] T007 [P] Define the training screening boundary and `TrainingUploadScreeningService` in `ContosoDashboard/Services/TrainingUploadScreeningService.cs`; always label results as simulated, never claim real malware detection, and fail closed for simulated-unsafe or incomplete outcomes.
- [ ] T008 Implement `LocalFileStorageService` in `ContosoDashboard/Services/LocalFileStorageService.cs`; store beneath the configured root outside `wwwroot`, generate GUID-based relative keys in `{userId}/{projectId or "personal"}/{guid}.{ext}` form, reject resolved paths outside the root, never use the uploaded name as a path, and support temporary-file promotion and cleanup.
- [ ] T009 Add Document, DocumentTag, and DocumentActivity sets, relationships, delete behaviors, and indexes to `ContosoDashboard/Data/ApplicationDbContext.cs`; retain activity records after document deletion and configure indexes for owner/upload date, project/upload date, category, normalized tag, and activity time/action.
- [ ] T010 Register `IFileStorageService`, `LocalFileStorageService`, and `TrainingUploadScreeningService` in `ContosoDashboard/Program.cs` using the configured local root; preserve local-only startup and the existing `EnsureCreated` schema behavior.

**Checkpoint**: All shared data and infrastructure compile; file storage stays private and screening is visibly a simulation.

---

## Phase 3: User Story 1 - Upload and Find Documents (Priority: P1, MVP)

**Goal**: Employees can upload allowed files with required metadata, receive per-file outcomes, and find only documents they may access.

**Independent Test**: Upload a valid file and locate it in My Documents; verify over-limit, unsupported, simulated-unsafe, incomplete-screening, path traversal, and unauthorized-list/content cases are rejected.

### Tests for User Story 1

- [ ] T011 [P] [US1] Add `DocumentServiceUploadTests` in `ContosoDashboard.Tests/Services/DocumentServiceUploadTests.cs` for required title/category, the exact six category values, optional description/project/tags, allowed file types, 26,214,400-byte maximum, per-file results, simulated-safe/unsafe/incomplete screening, and cleanup when metadata persistence fails.
- [ ] T012 [P] [US1] Add `LocalFileStorageServiceTests` in `ContosoDashboard.Tests/Services/LocalFileStorageServiceTests.cs` for GUID key generation, `{userId}/{projectId or "personal"}/{guid}.{ext}` layout, unique keys, storage-root containment, traversal rejection, and temporary-file promotion/cleanup.
- [ ] T013 [P] [US1] Add `DocumentContentEndpointTests` in `ContosoDashboard.Tests/Endpoints/DocumentContentEndpointTests.cs` for owner/project/department/admin access, denied and stale access, same not-found result for unauthorized and nonexistent IDs, and inline-versus-attachment type rules.

### Implementation for User Story 1

- [ ] T014 [US1] Define `IDocumentService` in `ContosoDashboard/Services/IDocumentService.cs` and implement `UploadAsync` in `ContosoDashboard/Services/DocumentService.cs`; validate title, category, supported extension/content type, and 26,214,400-byte maximum, screen before availability, save bytes before metadata, delete the new file if metadata persistence fails, and register `IDocumentService` in `ContosoDashboard/Program.cs`.
- [ ] T015 [US1] Implement My Documents listing, authorized search, sort, and filters in `ContosoDashboard/Services/DocumentService.cs` for title, description, tags, uploader, project, category, date range, and title/date/category/size ordering; apply authorization before returning or paging results.
- [ ] T016 [US1] Add and map the protected `GET /documents/{documentId:int}/content?disposition=inline|attachment` route in `ContosoDashboard/Endpoints/DocumentContentEndpoints.cs` and `ContosoDashboard/Program.cs`; re-check access on every request, return the same not-found response for unauthorized and missing documents, allow inline only for PDF/JPEG/PNG, set `X-Content-Type-Options: nosniff`, and never expose a local path.
- [ ] T017 [US1] Build `/documents` upload and My Documents flows in `ContosoDashboard/Pages/Documents.razor`, including multi-file selection, per-file title/category/optional metadata, progress, simulated-screening label, distinct outcomes, and list/search/sort/filter/empty/error states.
- [ ] T018 [US1] Add the Documents navigation link in `ContosoDashboard/Shared/NavMenu.razor` and verify it remains inside the authenticated application shell.
- [ ] T019 [US1] Record upload, preview, and download activity from `ContosoDashboard/Services/DocumentService.cs` and the protected content route, with actor/document/time and no file content or local path in activity details.

**Checkpoint**: US1 works locally and independently; an employee can upload and retrieve only authorized documents.

---

## Phase 4: User Story 2 - Use Project and Task Documents (Priority: P2)

**Goal**: Project members can use project documents; task views attach related documents; the dashboard shows recent documents and count; project additions notify members.

**Independent Test**: Associate documents with a project and task, confirm current members can access them and non-members cannot, then verify notifications and dashboard data for the signed-in user.

### Tests for User Story 2

- [ ] T020 [P] [US2] Add `DocumentServiceProjectTaskTests` in `ContosoDashboard.Tests/Services/DocumentServiceProjectTaskTests.cs` for project manager/member access, denied non-member access, task authorization, task/document project matching, project-member notifications excluding the uploader, and document association with the task's project.
- [ ] T021 [P] [US2] Add dashboard query tests in `ContosoDashboard.Tests/Services/DocumentDashboardTests.cs` verifying the signed-in user's document count and five most recent uploads without including another user's private documents.

### Implementation for User Story 2

- [ ] T022 [US2] Extend `DocumentService` in `ContosoDashboard/Services/DocumentService.cs` with project/task document queries and upload association; require current project manager/member or task access and reject a task/document project mismatch.
- [ ] T023 [US2] Add project-document notification types in `ContosoDashboard/Models/Notification.cs` and notify current project members through `ContosoDashboard/Services/NotificationService.cs` when a project document is added, excluding the uploader.
- [ ] T024 [P] [US2] Add project document listing and project-manager upload entry points to `ContosoDashboard/Pages/ProjectDetails.razor`, using the authorized DocumentService operations.
- [ ] T025 [P] [US2] Add document attachment/view actions to the task flow in `ContosoDashboard/Pages/Tasks.razor`; ensure uploaded task documents inherit the task's project and only authorized task users can access them.
- [ ] T026 [P] [US2] Extend `DashboardSummary` and its query in `ContosoDashboard/Services/DashboardService.cs` with the signed-in user's document count and five newest uploads.
- [ ] T027 [US2] Render the Recent Documents widget and document count card in `ContosoDashboard/Pages/Index.razor`, with loading, empty, and populated states based only on the current user's data.

**Checkpoint**: Project/task access and dashboard integration work independently of document sharing and reporting.

---

## Phase 5: User Story 3 - Manage, Preview, and Share Documents (Priority: P2)

**Goal**: Owners edit and replace their documents, owners/project managers delete only within permitted scope, and owners share with selected users or departments.

**Independent Test**: Verify owner edit/replacement/share, project-manager deletion within a managed project, denied manager edit/replacement, department recipient visibility, notification delivery, and unrelated-user denial.

### Tests for User Story 3

- [ ] T028 [P] [US3] Add `DocumentServiceManagementTests` in `ContosoDashboard.Tests/Services/DocumentServiceManagementTests.cs` for owner-only metadata edit/replacement, owner deletion, project-manager project-scoped deletion, department/user shares, duplicate grants, current-department access, notifications, and denied unrelated access.
- [ ] T029 [P] [US3] Add `DocumentShareTests` in `ContosoDashboard.Tests/Models/DocumentShareTests.cs` for `DocumentShareId: Integer, primary key`; `DocumentId: Integer, required FK to Document`; `RecipientUserId: Nullable integer FK to User`; `Department: Nullable string, max 100`; `SharedByUserId: Integer, required FK to User`; `SharedAtUtc: UTC date/time, required`; and the exactly-one-recipient constraint.

### Implementation for User Story 3

- [ ] T030 [US3] Add `DocumentShare` in `ContosoDashboard/Models/DocumentShare.cs` with `DocumentShareId: Integer, primary key`; `DocumentId: Integer, required FK to Document`; `RecipientUserId: Nullable integer FK to User`; `Department: Nullable string, max 100`; `SharedByUserId: Integer, required FK to User`; and `SharedAtUtc: UTC date/time, required`; enforce exactly one recipient user or department and prevent duplicate grants.
- [ ] T031 [US3] Add the DocumentShare set, relationships, exactly-one-recipient constraint, and recipient indexes to `ContosoDashboard/Data/ApplicationDbContext.cs`; department shares must resolve against current `User.Department`, separately from project membership.
- [ ] T032 [US3] Implement owner-only metadata edits and file replacement plus owner/project-manager scoped permanent deletion in `ContosoDashboard/Services/DocumentService.cs`; stage replacements before metadata update, keep the old file if the update fails, clean up the old file after success, and retain deletion activity.
- [ ] T033 [US3] Implement individual-user and selected-department sharing in `ContosoDashboard/Services/DocumentService.cs`; require document ownership, grant no access beyond selected recipients, recalculate department membership from current user data, and create per-recipient in-app notifications through the existing `INotificationService` plus activity records.
- [ ] T034 [US3] Add edit, replace, confirmed-delete, share, Shared with Me, and PDF/image preview controls to `ContosoDashboard/Pages/Documents.razor`; hide unavailable actions and show permission-aware success/error states.

**Checkpoint**: Ownership, project-manager deletion, department sharing, previews, and notifications follow the contract without broadening access.

---

## Phase 6: User Story 4 - Review Document Activity (Priority: P3)

**Goal**: Administrators review recorded document events and generate type, uploader, and access-pattern reports.

**Independent Test**: Create upload/download/preview/edit/replace/share/delete activity, confirm each event is attributed, and verify only administrators can view all three report summaries.

### Tests for User Story 4

- [ ] T035 [P] [US4] Add `DocumentActivityReportTests` in `ContosoDashboard.Tests/Services/DocumentActivityReportTests.cs` for upload/type counts, active uploader ordering, access patterns, and deletion snapshots after document removal.
- [ ] T036 [P] [US4] Add `DocumentReportsAuthorizationTests` in `ContosoDashboard.Tests/Pages/DocumentReportsAuthorizationTests.cs` confirming administrators can view reports and non-administrators cannot access the page directly or through navigation.

### Implementation for User Story 4

- [ ] T037 [US4] Implement administrator-only report queries in `ContosoDashboard/Services/DocumentService.cs` for most uploaded file types, most active uploaders, and document access patterns, using activity records without exposing file content or local paths.
- [ ] T038 [US4] Add an administrator-authorized reports page in `ContosoDashboard/Pages/DocumentReports.razor` for the three required report views, with empty and populated states.
- [ ] T039 [US4] Add an administrator-only Document Reports link in `ContosoDashboard/Shared/NavMenu.razor` and verify direct navigation is also denied to non-administrators.

**Checkpoint**: Administrators can report on retained activity, including deletion events, while other users cannot access the reports.

---

## Phase 7: Polish and Cross-Cutting Concerns

**Purpose**: Document training limitations, validate performance/security targets, and run the complete verification path.

- [ ] T040 [P] Update `README.md` with the offline local document root, supported file types/25 MiB limit, simulated-screening limitation, private content access, and non-production warning.
- [ ] T041 Update `specs/001-document-management/quickstart.md` with final setup/reset instructions and runnable test commands; never instruct users to delete the shared LocalDB instance or reset data without a backup.
- [ ] T042 Verify the list/search, upload, preview, and access-denial targets in `specs/001-document-management/quickstart.md`; record measured results for 500-document list/search, 25 MiB upload, and preview timing.
- [ ] T043 Run `dotnet restore ContosoDashboard.Tests/ContosoDashboard.Tests.csproj`, `dotnet build ContosoDashboard/ContosoDashboard.csproj`, `dotnet test ContosoDashboard.Tests/ContosoDashboard.Tests.csproj`, and all manual scenarios in `specs/001-document-management/quickstart.md`; record any unavailable checks and outcomes there.

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; creates the test project and storage-root configuration.
- **Foundational (Phase 2)**: Depends on Setup; blocks all stories because every story uses the shared document model, database setup, storage, and screening services.
- **User Story 1 (Phase 3)**: Depends on Foundational; MVP upload, list, search, and protected-content slice.
- **User Story 2 (Phase 4)**: Depends on US1 because project/task documents reuse upload, listing, and protected content.
- **User Story 3 (Phase 5)**: Depends on US1 and uses project context from US2 where applicable; implement after US2 to avoid concurrent edits to `DocumentService.cs` and `Documents.razor`.
- **User Story 4 (Phase 6)**: Depends on US1–US3 because reports consume their activity events.
- **Polish (Phase 7)**: Depends on all desired stories.

### User Story Dependencies

- **US1 (P1)**: Starts after Foundational; no dependency on other stories.
- **US2 (P2)**: Starts after US1; needs the core upload/list/content service and integrates project/task/dashboard surfaces.
- **US3 (P2)**: Starts after US1 and is sequenced after US2 to avoid shared-file conflicts; department sharing is independent of project membership.
- **US4 (P3)**: Starts after US1–US3 so all required activity types are available for reports.

### Parallel Opportunities

- Phase 1: T001 and T002 modify different files and can run in parallel.
- Phase 2: T003–T007 create independent model/interface files; T008 follows T006, T009 follows model tasks, and T010 follows storage/screening registration targets.
- US1: T011, T012, and T013 use separate test files and can be authored in parallel before their implementations.
- US2: T024, T025, and T026 touch separate UI/service files after T022/T023; T027 follows T026.
- US3: T028 and T029 write separate test files and can run together; implementation is sequential because it updates shared model/service/UI files.
- US4: T035 and T036 write separate test files and can run together; the service, page, and navigation work follows both tests.

---

## Parallel Examples

### User Story 1

```text
T011 DocumentService upload/screening tests in ContosoDashboard.Tests/Services/DocumentServiceUploadTests.cs
T012 Local storage safety tests in ContosoDashboard.Tests/Services/LocalFileStorageServiceTests.cs
T013 Content authorization tests in ContosoDashboard.Tests/Endpoints/DocumentContentEndpointTests.cs
```

### User Story 2

```text
T024 Project document UI in ContosoDashboard/Pages/ProjectDetails.razor
T025 Task document UI in ContosoDashboard/Pages/Tasks.razor
T026 Dashboard data query in ContosoDashboard/Services/DashboardService.cs
```

### User Story 3

```text
T028 Management tests in ContosoDashboard.Tests/Services/DocumentServiceManagementTests.cs
T029 Share-constraint tests in ContosoDashboard.Tests/Models/DocumentShareTests.cs
```

### User Story 4

```text
T035 Report query tests in ContosoDashboard.Tests/Services/DocumentActivityReportTests.cs
T036 Report page authorization tests in ContosoDashboard.Tests/Pages/DocumentReportsAuthorizationTests.cs
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational.
3. Complete Phase 3: User Story 1.
4. Verify the upload/list/search/content access flow with the US1 tests and independent manual checks.
5. Stop for validation before starting project/task integration or sharing.

### Incremental Delivery

1. Setup and foundational storage/model work.
2. US1: personal document upload and discovery MVP.
3. US2: project/task documents, notifications, dashboard integration.
4. US3: edit/replace/delete/share and preview controls.
5. US4: administrator activity reports.
6. Polish: documentation, performance checks, complete build/test/manual verification.

---

## Notes

- `[P]` tasks touch different files and have no dependency on unfinished tasks.
- Story labels map directly to the user stories in `specs/001-document-management/spec.md`.
- Entity field limits and nullability are specified verbatim in their model tasks and in `data-model.md`.
- All access decisions are enforced by `DocumentService` and rechecked for content requests; UI visibility alone is not authorization.
- Training screening is simulated and does not detect actual malware.
