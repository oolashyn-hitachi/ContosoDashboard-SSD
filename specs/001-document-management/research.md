# Research: Document Upload and Management

**Feature**: `001-document-management`
**Date**: 2026-10-05

## Decisions

### Local file storage and protected access

- **Decision**: Keep document bytes in a local storage root outside `wwwroot`, accessed only
  through an `IFileStorageService` implementation. Persist relative keys in the form
  `{userId}/{projectId or "personal"}/{guid}.{ext}` and resolve every key beneath the configured
  storage root before file access. The GUID filename is opaque and never uses the uploaded name.
  Serve previews and downloads through authenticated application endpoints that authorize the
  requesting user for the specific document; never return a filesystem path or static URL.
- **Rationale**: This meets the offline training constraint and prevents direct web access to files.
  The stakeholder requirements specify local storage outside the public web root and unique names.
- **Alternatives considered**: Storing files in `wwwroot` was rejected because it bypasses
  per-document authorization. Cloud storage was rejected because it violates the initial offline
  scope.

### Upload validation and screening

- **Decision**: Validate the allowed file types and 25 MiB (26,214,400-byte) size limit before storage. Route the
  upload through a training-only screening abstraction whose UI and result explicitly state that
  no real malware detection occurs. A simulated unsafe or incomplete result must prevent the file
  from becoming available. Use controllable test doubles for unsafe and incomplete outcomes.
- **Rationale**: The clarified requirement selects a training simulation, not antivirus software.
  Explicit labeling avoids implying protection the application does not provide, in line with the
  constitution.
- **Alternatives considered**: A real local scanner or cloud scanning service was rejected because
  neither is part of the offline training scope. Allowing uploads to bypass the simulation was
  rejected because every upload must have a visible screening outcome.

### Metadata and sharing model

- **Decision**: Use integer document keys and text categories. Apply the 25 MiB (26,214,400-byte)
  limit consistently with common browser upload-size conventions. Keep MIME type
  capacity at 255 characters. Represent tags as child records so each tag is searchable without delimiter parsing.
  Model a share as exactly one recipient user or one department name. Project membership stays a
  separate access grant based on the document's project association. Require a task-associated
  document's project, when present, to match the task's project.
- **Rationale**: The key and category formats match the stakeholder constraints and existing
  integer-key entities. Department sharing reuses the existing `User.Department` field and the
  current “My Team” grouping; no new group directory is introduced.
- **Alternatives considered**: Treating project members as department teams was rejected by the
  clarification. A decimal 25,000,000-byte limit was rejected in favor of the conventional 25 MiB
  browser limit. Storing tags as a delimited string was rejected because it complicates exact tag
  filtering and indexing. A new team entity was rejected as unnecessary scope.

### Authorization and activity records

- **Decision**: Centralize document queries and mutations in `DocumentService`; every operation
  receives the requesting user ID and evaluates ownership, role, current department membership,
  project membership, or explicit share access before returning metadata or content. Only owners
  edit metadata or replace files; project managers may delete documents in projects they manage.
  Add a
  `DocumentActivity` record for required events and content access. Preserve deletion activity when
  document metadata and bytes are permanently removed by retaining a document ID and title snapshot
  without a cascading foreign key to the deleted document.
- **Rationale**: Existing `ProjectService` and `TaskService` perform resource checks in their
  service methods, and the constitution requires authorization and data isolation at the service
  boundary. Activity data is needed for the specified reports.
- **Alternatives considered**: Relying only on Razor page authorization was rejected because it
  would not protect document content routes or direct service calls. Removing audit records with a
  document was rejected because deletion and access reports must remain available.

### File and database consistency

- **Decision**: Generate the storage key before writing. Save to a temporary file and move it into
  place, then persist metadata. If metadata persistence fails, remove the newly written file. For a
  replacement, write the new file first, commit metadata pointing to it, then remove the old file;
  report and log cleanup failures without exposing either path.
- **Rationale**: Filesystem writes and SQL transactions cannot share one atomic transaction. This
  ordering avoids database rows that point to files that were never saved and limits orphaned
  files through compensating cleanup.
- **Alternatives considered**: Persisting metadata before writing the file was rejected because a
  failed write leaves an unusable record. Replacing the old file in place was rejected because a
  failed write could destroy the current version.

### Schema initialization and local upgrade path

- **Decision**: Keep the repository's current `EnsureCreated` training setup and document that an
  existing LocalDB created before the new document entities must be backed up and recreated to
  receive the expanded schema. Do not silently delete a user's local database. Include explicit
  setup and reset guidance in the quickstart.
- **Rationale**: The application currently has no migrations or migration baseline, and
  `EnsureCreated` does not evolve an existing schema. The repository is explicitly training-only;
  its stakeholder document already describes clean-database setup for testing. A non-destructive
  upgrade path would require a separate migration-baseline project.
- **Alternatives considered**: Adding a full EF Core migration/baseline conversion was deferred
  because it materially expands this offline training feature. Automatically dropping the database
  was rejected because it destroys local training data without consent.

### Verification approach

- **Decision**: Add a focused .NET 8 test project using xUnit and an in-memory relational provider
  for document service authorization and metadata behavior, plus temporary-directory tests for
  local storage. Provide manual end-to-end checks for Blazor upload, preview/download, notifications,
  and dashboard integration.
- **Rationale**: The repository has no solution or test project. Access-control, filesystem, and
  metadata behavior carry enough risk to justify a small test harness; manual UI checks cover the
  existing Blazor Server workflow.
- **Alternatives considered**: Manual testing alone was rejected because the constitution requires
  automated tests for security, isolation, and persistence when a harness is available. Testing
  against SQL Server LocalDB alone was rejected because it reduces test portability.

## Repository Evidence

- The app targets .NET 8 and uses ASP.NET Core Blazor Server with EF Core 8 and SQL Server LocalDB
  (`ContosoDashboard/ContosoDashboard.csproj`, `ContosoDashboard/Program.cs`).
- Startup calls `Database.EnsureCreated()`; no migration files or test project were found.
- Existing project and task service methods authorize access using user IDs, project manager/member
  relationships, task assignment, and task creator status.
- Department is a property on `User`; `UserService.GetTeamMembersAsync` and the Team page use it to
  define a user's team. `ProjectMember` is a distinct relationship.
- Notifications are created through `INotificationService`; the dashboard summary is provided by
  `IDashboardService` and consumed in `Pages/Index.razor`.
- `Pages/ProjectDetails.razor` owns project details and tasks. `Pages/Tasks.razor` has a task-details
  action placeholder suitable for the task-document experience.