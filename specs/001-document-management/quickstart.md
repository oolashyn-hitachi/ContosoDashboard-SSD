# Quickstart: Document Upload and Management

This guide validates the offline training implementation. Simulated screening is not antivirus and
does not detect actual malware.

## Prerequisites

- .NET 8 SDK
- SQL Server LocalDB
- A Windows development environment with the ContosoDashboard repository
- An authenticated seeded user account from the mock login page

## Prepare the Local Database

The application creates the schema with EF Core `EnsureCreated`; it does not update an existing
database schema when entities change. Back up any local training data before resetting the
`ContosoDashboard` database. Delete only that database using SQL Server Object Explorer/SSMS or
`dotnet ef database drop --force` when the EF CLI is installed. Do not delete the shared LocalDB
instance. The app recreates and seeds the database at startup.

## Build and Automated Verification

From the repository root:

```powershell
dotnet restore ContosoDashboard/ContosoDashboard.csproj
dotnet build ContosoDashboard/ContosoDashboard.csproj
dotnet test ContosoDashboard.Tests/ContosoDashboard.Tests.csproj
dotnet run --project ContosoDashboard/ContosoDashboard.csproj
```

Expected: restore and build succeed, document service and storage tests pass, and the application
starts at the URL printed by `dotnet run`.

## Manual End-to-End Checks

1. Sign in as an employee and upload a PDF under 25 MiB with a title, category, tags, and no project.
   Confirm progress, the simulated-screening label, completion status, and the record in My Documents.
2. Try a file over 25 MiB and an unsupported extension. Confirm each is rejected and not listed.
   Use a test double in automated tests to verify simulated-unsafe and incomplete screening outcomes.
3. Search by title, description, tag, uploader, and project; sort and filter by the supported fields.
   Confirm another user's private document is absent from all results.
4. Upload a project document as a project member. Confirm another current project member can view
   and download it, while a non-member cannot retrieve its content route.
5. Share a personal document with one user, then with a department. Confirm only the selected
   recipient(s) can access it, Shared with Me and in-app notification update, and department access
   follows the recipient's current department.
6. As owner, exercise metadata edits, file replacement, and confirmed deletion. As a project
   manager, confirm deletion is allowed only for a document in a managed project and that editing
   or replacing another user's document is denied. Confirm deletion leaves an activity record but
   no document bytes or metadata row.
7. Attach a document to a task and confirm the task's project is associated. View it from the task
   and project, and confirm unauthorized users cannot retrieve it.
8. Confirm the dashboard shows the signed-in user's five most recent uploads and correct document
   count. Confirm project-addition and share notifications are delivered to the intended users.
9. As an administrator, generate document type, active uploader, and access-pattern reports and
   verify each includes the test activities.
10. Preview a PDF/image inline; download a text or Office file as an attachment. Confirm content
    responses do not reveal local storage paths and unauthorized requests return not found.