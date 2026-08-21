# Manually clean up PR preview folders in Azure Web App

This runbook explains how to remove old preview folders from Azure Web App by using the Azure interface.

Use this process because the release workflow does not clean old preview folders.
That behavior protects active previews, but it means stale folders accumulate over time.

Resource Group: `rg-eucs-eud-site`
Auth Requirements: PIM into Contributor via Entra for access

## When to run cleanup

Run this cleanup when one or more of these conditions are true:

- storage usage is increasing because of old preview content
- many closed pull requests still have `pr/<number>/` folders
- you are preparing for a maintenance window or content audit

Run cleanup on a regular cadence.
A monthly check is usually enough for this repository.

## Understand what you are deleting

Preview content is published under:

```text
/home/site/wwwroot/pr/<pull-request-number>/
```

Deleting a folder removes that preview environment.
It does not affect production content at `/home/site/wwwroot` unless you delete the wrong path.

> **Security note**
>
> Only users with the correct Azure RBAC (role-based access control, Azure authorization model) role should perform this task.
> Use least privilege and follow your team change controls.

## Before you start

Before deleting anything:

1. List candidate PR folders to remove.
2. Confirm each related pull request is closed or merged.
3. Confirm no active testing depends on that preview URL.

## Clean up using Azure Portal and Kudu

### Open the App Service

1. Sign in to Azure Portal.
2. Open the target App Service instance.
3. Check you are in the correct subscription and resource group.

### Open Advanced Tools (Kudu)

1. In the App Service menu, go to Development Tools.
2. Select Advanced Tools.
3. Select Go to open the Kudu site.

Kudu is the App Service management site used for file operations and deployment diagnostics.

### Navigate to the preview folder root

1. In Kudu, open Debug Console.
2. Choose Bash.
3. Navigate to `/home/site/wwwroot/pr/`.

### Delete stale PR folders

Use one of these methods:

- File Manager: select stale `pr/<number>` folders and delete them.
- Bash console: remove selected folders with shell commands.

If you use Bash, delete only explicit folder names.
Do not run broad wildcard deletes unless the team has reviewed the exact command.

Example safe pattern:

```bash
rm -rf /home/site/wwwroot/pr/123
rm -rf /home/site/wwwroot/pr/456
```

### Validate after deletion

1. Refresh the Kudu file view and confirm the folders are gone.
2. Open one deleted preview URL and confirm it no longer serves content.
3. Open the main site URL and confirm production content still works.

## Troubleshooting

### You cannot open Kudu

Check:

- your Azure RBAC assignment
- App Service access restrictions
- whether the App Service is running

### Deletion fails in File Manager

Try the Bash console in Kudu.
If files are locked, retry after a short pause.

### A deleted preview URL still works

Confirm you deleted the correct folder number.
Then clear browser cache and check again.

If content still appears, verify whether another deployment recreated the folder.

## Team guardrails

Follow these guardrails every time:

1. Never delete `/home/site/wwwroot` directly.
2. Only delete specific `pr/<number>` folders.
3. Do not run unreviewed bulk-delete commands.
4. Keep a simple audit note of what changed.

## Related runbooks

- [Publish to Azure Web App](./publish-to-azure-web-app.md)