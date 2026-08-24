# Publish to Azure Web App

This runbook explains how the site publishes to Azure Web App (Azure App Service, Microsoft Azure's managed web hosting service).

It covers the two GitHub Actions workflows, the Azure identity setup, and the runtime details that matter to developers and architects.

## When to use this runbook

Use this runbook when you need to:

- understand how preview and release deployments work
- troubleshoot a failed App Service deployment
- change the Azure hosting or authentication model
- explain the deployment architecture to another engineer or architect

## Deployment model

The repository now publishes in two paths at the same time.

- GitHub Pages still publishes the documentation site as before.
- Azure Web App publishes the same generated HTML, CSS, JavaScript, images, and supporting files.

The Azure flow does not replace GitHub Pages yet.
Both paths run side by side.

## Understand the workflows

### Preview deployments

The preview workflow runs on pull requests against `main`.

It does three things:

1. Builds the site into `artifact.tar`.
2. Packages the preview content under `pr/<pull-request-number>/` inside the ZIP archive.
3. Deploys that ZIP to Azure Web App by using `az webapp deploy`.

The preview URL follows this pattern:

```text
https://<app-name>.azurewebsites.net/pr/<pull-request-number>/index.html
```

The workflow also posts the preview URL back to the pull request.

### Release deployments

The publish workflow runs on merges to `main`.

It does two things:

1. Builds the site into `artifact.tar`.
2. Deploys the generated site to the root of Azure Web App.

This release path leaves `pr/<number>/` content in place.
It no longer tries to clean or delete preview folders.

## Understand the Azure identity

The deployment workflow uses OpenID Connect (OIDC, a federated login flow that avoids stored cloud credentials) through Azure Login.

The Entra ID application registration is `eucs-eud-site-infra`.

The workflow needs these secrets:

- `AZURE_APP_SERVICE_NAME`
- `AZURE_RESOURCE_GROUP_NAME`
- `ARM_CLIENT_ID`
- `ARM_TENANT_ID`
- `ARM_SUBSCRIPTION_ID`

The GitHub workflow uses those values to log in to Azure and call `az webapp deploy`.

> **Security note**
>
> Do not reintroduce SCM/Kudu publishing credentials into the workflow unless you have a clear need.
> The current flow uses Azure Login with OIDC, which avoids long-lived secrets for deployment.

## Understand the App Service runtime

The App Service runs on Linux with the Node 24 LTS stack.

App Service does not serve the static files by itself in this setup.
You must keep a startup command in place so the runtime serves the deployed files.

Use this startup command:

```bash
pm2 serve /home/site/wwwroot --no-daemon --spa --port 8080
```

This command:

- serves the deployed files from `/home/site/wwwroot`
- supports single-page application routing with `--spa`
- keeps the process in the foreground with `--no-daemon`

If the site returns 404 after a successful deployment, check the startup command first.

## Understand the packaging step

The Azure workflow downloads `artifact.tar`, unpacks it, and repackages the content as `site.zip`.

For preview deployments, the workflow places the content under `site-root/pr/<pull-request-number>/` before zipping it.

For release deployments, the workflow packages the site at the root of the ZIP archive.

This design matters because `az webapp deploy` now writes to the App Service root.
The ZIP layout controls whether the files end up under `pr/<number>/` or at the site root.

## Know the deployment command

The reusable workflow uses `az webapp deploy` with these important options:

- `--src-path site.zip`
- `--type zip`
- `--restart true`

The workflow no longer passes `--clean`.
It also no longer uses `--target-path`.

That means deployments leave existing files in place unless the ZIP overwrites them.

## Know the operational tradeoffs

### Leftover files stay in place

The release flow intentionally keeps leftover files.

That reduces the risk of preview folders disappearing during a main release.
It also means old files can remain if the build stops producing them.

If you need to remove obsolete content, do it deliberately.
Do not rely on the release pipeline to clean the target directory.

### Preview folders persist

Preview folders live under `pr/<number>/`.

The main release no longer deletes them.
This supports side-by-side preview and release content.

### GitHub Pages still runs

The GitHub Pages deployment remains active.

Do not remove it unless the project decides to retire Pages entirely.

## Troubleshoot deployment failures

### Check the latest deployment log

When Azure reports a deployment failure, inspect the latest deployment record in App Service.

Use the Azure portal or the deployment log API URL shown in the workflow failure.

Look for these common issues:

- invalid ZIP structure
- missing `artifact.tar`
- authentication or role assignment problems
- a bad App Service name or resource group

### Check the package structure

The ZIP file must contain the site files in the expected structure.

For previews, confirm that the package contains `pr/<pull-request-number>/index.html`.
For releases, confirm that `index.html` and the supporting assets sit at the ZIP root.

### Check the runtime startup command

If deployment succeeds but the site returns 404, confirm the startup command.

The App Service must run:

```bash
pm2 serve /home/site/wwwroot --no-daemon --spa --port 8080
```

### Check the workflow inputs

If a deployment fails early, check these values:

- `artifact_name`
- `deployment_action`
- `preview_subpath`
- `slot_name`

The reusable workflow only deploys when `deployment_action` is `deploy`.

## Make changes safely

If you need to change the Azure deployment flow, keep these rules in mind:

1. Keep preview and release publishing separate.
2. Keep GitHub Pages running until the team agrees to retire it.
3. Avoid reintroducing cleanup logic that can delete preview folders.
4. Preserve the App Service startup command for Linux.
5. Validate any workflow change with schema checks and `git diff --check`.

## Reference files

- [Preview workflow](../../.github/workflows/preview.yml)
- [Release workflow](../../.github/workflows/publish.yml)
- [Reusable App Service deploy workflow](../../.github/workflows/upload-preview-to-app-service.yml)