# github-actions
Centralised Github Workflow repository to remove duplicate workflow logic from repo level

## Workflow and action layers

Reusable workflows share job orchestration, while composite actions package related implementation steps. This keeps responsibilities in three layers:

```text
Application workflow → Reusable workflow → Composite action → GHCR
```

| Layer | Responsibility |
| --- | --- |
| Application workflow | Decides when to run and whether publishing must wait for tests or other jobs. |
| [Reusable workflow](.github/workflows/build-push-ghcr.yml) | Selects the runner, sets permissions, checks out the caller's source, and invokes the action. |
| [Composite action](.github/actions/build-push-ghcr/action.yml) | Sets up Buildx, logs in to GHCR, determines the image name and tags, and builds and pushes the image. |

For example, ModernBlog can require tests to pass by setting `needs: test` on the job that calls the reusable workflow. Other applications choose their own dependencies. Changes to image tagging belong in the composite action; changes to the runner or permissions belong in the reusable workflow.

Checkout stays in the reusable workflow so the action works with the source already in the workspace. The workflow explicitly passes `${{ github.token }}` through the action's `token` input for registry login; permissions remain a workflow concern.

## Build and push to GitHub Container Registry

The [`build-push-ghcr.yml`](.github/workflows/build-push-ghcr.yml) workflow builds a Docker image from a repository and uploads it to GitHub Container Registry (GHCR), where it can be downloaded and run later.

It is a reusable workflow: another repository's GitHub Actions workflow calls it. It does not trigger automatically on pushes; the calling workflow determines when it runs.

When called, the workflow checks out the source and delegates publishing to the composite action. Together they:

1. Checks out the calling repository's code.
2. Sets up Docker Buildx to build the image.
3. Logs in to GHCR using GitHub's automatic token, with permission to publish packages.
4. Chooses the image name. By default this is `ghcr.io/OWNER/REPOSITORY`; callers can supply the optional `image-name` input to override it.
5. Adds image tags:
   - The branch name for branch runs.
   - `latest` when running on the default branch.
   - `commit-<short-sha>` to identify the source commit.
6. Builds and pushes the image using the repository root as the build context and its default `Dockerfile`, targeting `linux/amd64`.

For example, a run on `main` for `acme/my-app` can publish the image with all of these tags:

```text
ghcr.io/acme/my-app:main
ghcr.io/acme/my-app:latest
ghcr.io/acme/my-app:commit-abc1234
```

The workflow publishes the image; it does not deploy or run the application. It also has no separate test step.
