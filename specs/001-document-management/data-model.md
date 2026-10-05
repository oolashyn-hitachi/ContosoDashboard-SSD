# Data Model: Document Upload and Management

## Entities

### Document

Represents one uploaded file and its searchable metadata.

| Field | Type / constraint | Rules |
|-------|-------------------|-------|
| DocumentId | Integer, primary key | Consistent with existing User, Project, and Task keys |
| Title | String, required, max 255 | Trimmed; non-empty |
| Description | Nullable string, max 2000 | Optional |
| Category | String, required | One of Project Documents, Team Resources, Personal Files, Reports, Presentations, Other |
| OriginalFileName | String, max 255 | Display metadata only; never used as a path |
| RelativeFilePath | String, required, max 512, unique | `{userId}/{projectId or "personal"}/{guid}.{ext}` under the configured local root; GUID filename only |
| FileSizeBytes | Long, required | Greater than zero and at most 26,214,400 bytes (25 MiB) |
| FileType | String, required, max 255 | Normalized MIME type; do not trust the browser value as proof of file safety |
| UploadedAtUtc | UTC date/time, required | Set by the service |
| UploadedByUserId | Integer, required FK to User | Owner/uploader |
| ProjectId | Nullable integer FK to Project | Set for project documents |
| TaskId | Nullable integer FK to Task | If set, its ProjectId must match the document's ProjectId |

### DocumentTag

One searchable custom tag assigned to a document.

| Field | Type / constraint | Rules |
|-------|-------------------|-------|
| DocumentTagId | Integer, primary key | |
| DocumentId | Integer, required FK to Document | Delete with the document |
| Value | String, required, max 100 | Trimmed; unique per document after normalization |

### DocumentShare

One explicit access grant on a document.

| Field | Type / constraint | Rules |
|-------|-------------------|-------|
| DocumentShareId | Integer, primary key | |
| DocumentId | Integer, required FK to Document | Delete with the document |
| RecipientUserId | Nullable integer FK to User | Set for an individual recipient |
| Department | Nullable string, max 100 | Set for a department recipient; matches current User.Department |
| SharedByUserId | Integer, required FK to User | Must be the document owner |
| SharedAtUtc | UTC date/time, required | Set by the service |

Exactly one of `RecipientUserId` or `Department` must be set. A department share grants access to
users currently in that department; project membership is evaluated separately and is not persisted
as a share. Duplicate grants to the same recipient are prevented.

### DocumentActivity

An append-only record used for audit and administrator reports. Supported actions include upload,
download, preview, metadata edit, file replacement, share, and permanent deletion.

| Field | Type / constraint | Rules |
|-------|-------------------|-------|
| DocumentActivityId | Integer, primary key | |
| DocumentId | Nullable integer, no cascading FK | Retained as an identifier after permanent deletion |
| DocumentTitleSnapshot | String, max 255 | Allows deletion activity to remain understandable |
| ActorUserId | Integer, required FK to User | User who performed the operation |
| Action | String or enum, required | One of the supported activity types |
| OccurredAtUtc | UTC date/time, required | Set by the service |
| Details | Nullable string, max 1000 | Non-sensitive summary; must not contain file content or local paths |

### Relationships

- User has many uploaded Documents; deleting a User is restricted while owned documents exist.
- Project has zero or more Documents; project managers and current project members receive
  project-scoped access according to the feature requirements.
- Task has zero or more Documents. A task document inherits the task's project association and
  access rules.
- Document has many DocumentTags, DocumentShares, and DocumentActivity entries.
- DocumentActivity is intentionally independent of the Document foreign-key lifecycle so permanent
  deletion does not erase its audit event.

## Validation and Indexes

- Enforce the fixed category list, supported extensions/types, non-empty title, tag length, and
  per-file size limit before storage and again at the service boundary.
- Generate storage paths as `{userId}/{projectId or "personal"}/{guid}.{ext}`; never concatenate an
  uploaded filename into a filesystem path. Verify the normalized resolved path stays under the
  storage root.
- Index document owner plus upload date, project plus upload date, category, normalized tag, and
  document activity time/action. Search and all returned rows must be authorization-filtered before
  paging or display.
- Use text storage for categories and a maximum 255-character FileType field.

## Lifecycle

1. Validate metadata, file size, and supported type; obtain an explicit simulated screening result.
2. A simulated unsafe or unavailable result ends the upload without making a document available.
3. For an accepted training-simulation result, write to a temporary local file and promote it to
   its generated relative path; then persist Document, tags, and upload activity.
4. If metadata persistence fails, remove the new file. For replacement, commit the new path before
   deleting the old file; keep the old path usable if the metadata update fails.
5. For permanent deletion, record the deletion event and title snapshot, remove the file and
   metadata after confirmation, and preserve the activity record.