# Scenario Instructions

## Scenario
- **Name**: .NET Version Upgrade
- **Solution**: `D:\a\dotnet-migration-copilot-samples\dotnet-migration-copilot-samples\ContosoUniversity\ContosoUniversity.sln`
- **Target Framework**: `net10.0` (.NET 10.0 LTS)

## Preferences

### Flow Mode
Mode: **Automatic** — Run end-to-end, only pause when blocked or needing user input.

### Technical Preferences
- Target framework: net10.0 (.NET 10.0 LTS)

## Source Control
- Working branch: `copilot/upgrade-solutions-to-dotnet-10-yet-again`
- Source branch: `main`

## Strategy
**Selected**: All-at-Once
**Rationale**: Single project solution; work organized by concern due to high complexity (MSMQ, System.Web, Legacy Config migration).

### Execution Constraints
- Single atomic upgrade — all projects updated together
- SDK-style conversion must happen before TFM change
- Validate full solution build after upgrade (0 errors)
- Testing comes after atomic upgrade completes successfully

## Preferences
- **Flow Mode**: Automatic
- **Commit Strategy**: Single Commit at End

## Decisions
- 2026-03-16: User requested upgrade to .NET 10. Automatic flow mode selected (default).
- 2026-03-16: All-at-Once strategy selected (single project edge case).
