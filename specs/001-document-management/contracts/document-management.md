# Document Management Contracts

This feature is internal to the Blazor Server application. It adds no public REST API or cloud
integration. UI events call the authorized document service; binary content uses the protected
application route below.

## User-Facing Routes

| Route / location | Contract |
|------------------|----------|
| `/documents` | Authenticated users can view My Documents and Shared with Me, upload files, search, sort, filter, edit metadata, replace files, share, and delete within their permissions. |
| `/projects/{projectId}` | The existing project details page shows project documents only to the project manager and current project members; project managers can add project documents and delete documents in projects they manage. |
| `/tasks` task details | The existing task view exposes related documents and upload/attach actions only to users authorized for the task; an uploaded task document inherits the task's project. |
| `/` dashboard | Shows the signed-in user's five newest uploads and their uploaded-document count. |

## Service Operations

Every operation uses the authenticated user's integer ID as the requester. Services must not accept
client-supplied owner IDs as proof of authorization.

| Operation | Required behavior |
|-----------|-------------------|
| List/search | Return only documents the requester may access; support the specified search fields, filters, and sort orders. |
| Upload | Accept one file plus title, category, optional description/project/task/tags; enforce size/type and simulated-screening outcomes; return a per-file success or clear rejection. |
| Edit/replace | Require document ownership; validate new metadata/file before replacing the current version. Project managers cannot edit or replace another user's document. |
| Share | Require ownership; target exactly one user or one department; notify recipients and create an activity entry. Project membership remains a separate access rule. |
| Delete | Require ownership or project-manager access for a document in a project they manage; require explicit confirmation in the UI; permanently remove the file and document metadata while retaining deletion activity. |
| Preview/download | Re-check current authorization for every request, record access activity, and stream only the requested file. |
| Report | Administrator-only summaries for most uploaded file types, active uploaders, and access patterns. |

## Local Storage Boundary

- `UploadAsync` stores bytes under the configured local root using the generated relative key
  `{userId}/{projectId or "personal"}/{guid}.{ext}` and returns that key for metadata persistence.
- `DeleteAsync` and `DownloadAsync` accept only stored relative keys and reject any resolved path
  outside the configured root.
- `GetUrlAsync` returns the protected application content route, never a disk path or static-file
  URL. The route re-checks current document authorization; URL generation itself grants no access.

## Protected Content Route

- `GET /documents/{documentId:int}/content?disposition=inline|attachment`
- Require authentication and document-level authorization on each request. Return the document bytes
  with the recorded content type. `inline` is permitted only for PDF and JPEG/PNG; other supported
  types use `attachment`.
- Return the same not-found response for nonexistent and unauthorized document IDs to avoid
  disclosing document existence. Never include a local path in a response.
- Send `X-Content-Type-Options: nosniff`; set a safe display filename from stored metadata. Do not
  trust a request-supplied path, MIME type, or filename.

## Authorization Outcomes

- Owner: manage own documents and create shares.
- Project manager: manage documents associated with projects they manage.
- Project member: view/download documents associated with projects they currently belong to.
- Department member: view/download only documents explicitly shared with their current department.
- Team lead: manage documents uploaded by users in their current department.
- Administrator: access all documents and reports.
- Any other requester: no metadata or file content for the document.

Overlapping grants are additive; removing one grant does not remove access supplied by another
current grant, such as project membership.