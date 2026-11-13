---
name: msstore-cli
description: 'Microsoft Store Developer CLI (msstore) for publishing Windows applications to the Microsoft Store. Use when asked to configure Store credentials, list Store apps, check submission status, publish submissions, manage package flights, set up CI/CD for Store publishing, or integrate with Partner Center. Supports Windows App SDK/WinUI, UWP, .NET MAUI, Flutter, Electron, React Native, and PWA applications.'
license: MIT
---

# Microsoft Store Developer CLI (msstore)

The Microsoft Store Developer CLI (`msstore`) is a cross-platform command-line interface for publishing and managing applications in the Microsoft Store. It integrates with Partner Center APIs and supports automated publishing workflows for various application types.

## When to Use This Skill

Use this skill when you need to:

- Configure Store credentials for API access
- List applications in your Store account
- Check the status of a submission
- Publish submissions to the Store
- Package applications for Store submission
- Initialize projects for Store publishing
- Manage package flights (beta testing)
- Set up CI/CD pipelines for automated Store publishing
- Manage gradual rollouts of submissions
- Update submission metadata programmatically

## Prerequisites

- Windows 10+, macOS, or Linux
- .NET 9 Desktop Runtime (Windows) or .NET 9 Runtime (macOS/Linux)
- Partner Center account with appropriate permissions
- Azure AD app registration with Partner Center API access
- msstore CLI installed via one of these methods:
  - **Microsoft Store**: [Download](https://www.microsoft.com/store/apps/9P53PC5S0PHJ)
  - **WinGet**: `winget install "Microsoft Store Developer CLI"`
  - **Manual**: Download from [GitHub Releases](https://aka.ms/msstoredevcli/releases)

### Partner Center Setup

Before using msstore, you need to create an Azure AD application with Partner Center access:

1. Go to [Partner Center](https://partner.microsoft.com/dashboard)
2. Navigate to **Account settings** > **User management** > **Azure AD applications**
3. Create a new application and note the **Tenant ID**, **Client ID**, and **Client Secret**
4. Grant the application appropriate permissions (Manager or Developer role)

## Core Commands Reference

### Command Reference

Full syntax, options, and examples for each `msstore` command. See [references/commands.md](references/commands.md) for:

- info - Print Configuration
- reconfigure - Configure Credentials
- settings - CLI Settings
- apps - Application Management
- submission - Submission Management
- init - Initialize Project for Store
- package - Package for Store
- publish - Publish to Store
- flights - Package Flight Management

## Common Workflows

### Workflow 1: First-Time Store Setup

```bash
# 1. Install the CLI
winget install "Microsoft Store Developer CLI"

# 2. Configure credentials (get these from Partner Center)
msstore reconfigure --tenantId $TENANT_ID --sellerId $SELLER_ID --clientId $CLIENT_ID --clientSecret $CLIENT_SECRET

# 3. Verify configuration
msstore info

# 4. List your apps to confirm access
msstore apps list
```

### Workflow 2: Initialize and Publish New App

```bash
# 1. Navigate to project
cd my-winui-app

# 2. Initialize for Store (creates/updates app identity)
msstore init .

# 3. Package the application
msstore package . --arch x64,arm64

# 4. Publish to Store
msstore publish .

# 5. Check submission status
msstore submission status <productId>
```

### Workflow 3: Update Existing App

```bash
# 1. Build your updated application
dotnet publish -c Release

# 2. Package and publish
msstore publish ./my-app

# Or publish from existing package
msstore publish ./my-app --inputFile ./artifacts/MyApp.msixupload
```

### Workflow 4: Gradual Rollout

```bash
# 1. Publish with initial rollout percentage
msstore publish ./my-app --packageRolloutPercentage 10

# 2. Monitor and increase rollout
msstore submission poll <productId>

# 3. (After validation) Finalize to 100%
# This completes via Partner Center or submission update
```

### Workflow 5: Beta Testing with Flights

```bash
# 1. Create a flight group in Partner Center first
# Then create a flight
msstore flights create <productId> "Beta Testers" --group-ids "group-id-1,group-id-2"

# 2. Publish to the flight
msstore publish ./my-app --flightId <flightId>

# 3. Check flight submission status
msstore flights submission status <productId> <flightId>

# 4. After testing, publish to production
msstore publish ./my-app
```

### Workflow 6: CI/CD Pipeline Integration

```yaml
# GitHub Actions example
name: Publish to Store

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9.0.x'
      
      - name: Install msstore CLI
        run: winget install "Microsoft Store Developer CLI" --accept-package-agreements --accept-source-agreements
      
      - name: Configure Store credentials
        run: |
          msstore reconfigure --tenantId ${{ secrets.TENANT_ID }} --sellerId ${{ secrets.SELLER_ID }} --clientId ${{ secrets.CLIENT_ID }} --clientSecret ${{ secrets.CLIENT_SECRET }}
      
      - name: Build application
        run: dotnet publish -c Release
      
      - name: Publish to Store
        run: msstore publish ./src/MyApp
```

## Integration with winapp CLI

The winapp CLI (v0.2.0+) integrates with msstore via the `winapp store` subcommand:

```bash
# These commands are equivalent:
msstore reconfigure --tenantId xxx --clientId xxx --clientSecret xxx
winapp store reconfigure --tenantId xxx --clientId xxx --clientSecret xxx

# List apps
msstore apps list
winapp store apps list

# Publish
msstore publish ./my-app
winapp store publish ./my-app
```

Use `winapp store` when you want a unified CLI experience for both packaging and publishing.

## Troubleshooting

| Issue | Solution |
| ----- | -------- |
| Authentication failed | Verify credentials with `msstore info`; re-run `msstore reconfigure` |
| App not found | Ensure the product ID is correct; run `msstore apps list` to verify |
| Insufficient permissions | Check Azure AD app role in Partner Center (needs Manager or Developer) |
| Package validation failed | Ensure package meets Store requirements; check Partner Center for details |
| Submission stuck | Run `msstore submission poll <productId>` to check status |
| Flight not found | Verify flight ID with `msstore flights list <productId>` |
| Rollout percentage invalid | Value must be between 0 and 100 |
| Init fails for PWA | Ensure URL is publicly accessible and has valid web app manifest |

## Environment Variables

The CLI supports environment variables for credentials:

| Variable | Description |
| -------- | ----------- |
| `MSSTORE_TENANT_ID` | Azure AD Tenant ID |
| `MSSTORE_SELLER_ID` | Partner Center Seller ID |
| `MSSTORE_CLIENT_ID` | Azure AD Application Client ID |
| `MSSTORE_CLIENT_SECRET` | Client Secret |

## References

- [Microsoft Store Developer CLI Documentation](https://learn.microsoft.com/windows/apps/publish/msstore-dev-cli/overview)
- [CLI Commands Reference](https://learn.microsoft.com/windows/apps/publish/msstore-dev-cli/commands)
- [GitHub Repository](https://github.com/microsoft/msstore-cli)
- [Partner Center API](https://learn.microsoft.com/windows/uwp/monetize/using-windows-store-services)
- [App Submission API](https://learn.microsoft.com/windows/uwp/monetize/create-and-manage-submissions-using-windows-store-services)
- [Package Flights Overview](https://learn.microsoft.com/windows/uwp/publish/package-flights)
- [Gradual Package Rollout](https://learn.microsoft.com/windows/uwp/publish/gradual-package-rollout)
