# github-actions
Centralised Github Workflow repository to remove duplicate workflow logic from repo level

## Build and push to GitHub Container Registry

The [`build-push-ghcr.yml`](.github/workflows/build-push-ghcr.yml) workflow builds a Docker image from a repository and uploads it to GitHub Container Registry (GHCR), where it can be downloaded and run later.

It is a reusable workflow: another repository's GitHub Actions workflow calls it. It does not trigger automatically on pushes; the calling workflow determines when it runs.

When called, the workflow:

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
