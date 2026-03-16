# .NET Version Upgrade Plan

## Overview

**Target**: Upgrade ContosoUniversity from .NET Framework 4.8 (net48) to .NET 10.0
**Scope**: 1 project (~3,400 LOC), non-SDK-style WAP with significant API breaking changes (MSMQ, System.Web, Legacy Configuration)

### Selected Strategy
**All-At-Once** — Single project upgraded in one operation, organized by concern.
**Rationale**: 1 project. Work is organized by concern due to high complexity (MSMQ migration, System.Web migration, Legacy Configuration migration, non-SDK project format).

---

## Tasks

### 01-sdk-conversion: Convert project to SDK-style

The ContosoUniversity.csproj is a legacy non-SDK-style Web Application Project (WAP) targeting .NET Framework 4.8. It must be converted to SDK-style format before the target framework can be changed to net10.0.

This involves replacing the legacy project file structure with a modern SDK-style `<Project Sdk="Microsoft.NET.Sdk.Web">` format, removing the old project type GUIDs, removing redundant file includes, and cleaning up imports that are now handled by the SDK.

**Done when**: The project file uses SDK-style format (`<Project Sdk="Microsoft.NET.Sdk.Web">`), the solution loads and restores without errors.

---

### 02-packages: Update NuGet packages

Update all NuGet packages to versions compatible with net10.0. Key changes include:

- Remove packages whose functionality is now included in the framework (Microsoft.AspNet.Mvc, Microsoft.AspNet.Razor, Microsoft.AspNet.WebPages, Microsoft.CodeDom.Providers.DotNetCompilerPlatform, NETStandard.Library, Microsoft.Web.Infrastructure, System.Buffers, System.Memory, System.Numerics.Vectors, System.Threading.Tasks.Extensions, System.ComponentModel.Annotations)
- Remove the incompatible package: Microsoft.AspNet.Web.Optimization (bundling not supported in .NET Core)
- Remove the deprecated package: Microsoft.Identity.Client (need replacement or removal)
- Fix the security vulnerability: Microsoft.Data.SqlClient (2.1.4 → 6.1.4)
- Replace ANTLR 3.x with Antlr4 4.6.6 (or remove if unused)
- Upgrade Microsoft.EntityFrameworkCore and all related packages (3.1.32 → 10.0.x)
- Upgrade Microsoft.Extensions.* packages (3.1.32 → 10.0.x)
- Upgrade Newtonsoft.Json (13.0.3 → 13.0.4)
- Remove packages that will be replaced during code migration (System.Messaging, WebGrease)

**Done when**: All package references compile without incompatibility errors; `dotnet restore` succeeds with no warnings about unsupported packages; security vulnerabilities resolved.

---

### 03-code-migration: Migrate incompatible APIs and application code

Fix all API incompatibilities identified in the assessment. Three main areas:

**1. ASP.NET Framework (System.Web) migration** — 16 issues
Replace System.Web APIs with ASP.NET Core equivalents. Key changes:
- Migrate `HttpPostedFileBase` file upload handling to `IFormFile`
- Replace `System.Web.Routing.RouteCollection` routing with ASP.NET Core route registration (MapControllerRoute, etc.)
- Replace `System.Web.HttpApplication` (Global.asax.cs) with `Program.cs` / middleware pipeline

**2. MSMQ migration** — 59 issues
`System.Messaging` (MSMQ) APIs are not supported in .NET Core. Replace with the MSMQ NuGet package for .NET (`Particular.Msmq`) or migrate to a modern message queue. The Services directory likely contains the MSMQ usage.

**3. Legacy Configuration System migration** — 16 issues
Replace `System.Configuration.ConfigurationManager` (app.config/web.config) with `Microsoft.Extensions.Configuration`. Key changes:
- Replace `ConfigurationManager.AppSettings` reads with `IConfiguration`
- Replace `ConfigurationManager.ConnectionStrings` with connection string configuration
- Migrate web.config connection strings to appsettings.json

**4. Application initialization** (Global.asax.cs → Program.cs)
Convert application startup code from Global.asax.cs to ASP.NET Core's `Program.cs` and middleware pipeline.

**Done when**: Solution builds with 0 errors; all replaced APIs use .NET Core equivalents; no references to System.Web, System.Messaging (or replaced with compatible package), System.Configuration.

---

### 04-validation: Build and test validation

Validate the complete upgrade by building the solution and running any existing tests.

- Run `dotnet build` and confirm 0 errors, acceptable warnings
- Run `dotnet test` if test projects exist
- Verify application starts (if applicable)
- Review any remaining build warnings for potential runtime issues

**Done when**: Solution builds with 0 errors; all tests pass (or pre-existing test failures are documented).
