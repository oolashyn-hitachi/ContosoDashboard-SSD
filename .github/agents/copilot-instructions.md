# oolashyn-hitachi-automatic-tribble Development Guidelines

Auto-generated from all feature plans. Last updated: 2026-10-05

## Active Technologies
- C# on .NET 8 + ASP.NET Core 8 Blazor Server; EF Core 8 SQL Server; xUnit; EF Core SQLite (001-document-management)
- SQL Server LocalDB for metadata; local filesystem outside `wwwroot` for document bytes (001-document-management)

## Project Structure

```text
ContosoDashboard/
├── Data/
├── Models/
├── Pages/
├── Services/
├── Shared/
└── wwwroot/
ContosoDashboard.Tests/ (planned)
specs/001-document-management/
```

## Commands

```powershell
dotnet restore ContosoDashboard/ContosoDashboard.csproj
dotnet build ContosoDashboard/ContosoDashboard.csproj
dotnet test ContosoDashboard.Tests/ContosoDashboard.Tests.csproj
dotnet run --project ContosoDashboard/ContosoDashboard.csproj
```

## Code Style

C# on .NET 8: Follow standard conventions

## Recent Changes
- 001-document-management: Added C# on .NET 8 + ASP.NET Core 8 Blazor Server; EF Core 8 SQL Server; xUnit; EF Core SQLite

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
