Dockerfiles for https://hub.docker.com/u/developmentrunsk and mostly used by https://github.com/RunDevelopmentSk/devcontainers.

All build sources (the `Dockerfile.*` files and the `.sh` scripts) live in the `images/` directory, keeping the repository root reserved for documentation and tooling configuration.

The naming convention is as follows (`images/Dockerfile.<image_label>_<image_tag>`):
- if `<image_label>` starts with `fajn` or `dev-`, it is a Dockerfile intended for project development in a devcontainer.
- if `<image_label>` starts with `prod-`, it is for production project execution

Images are built and pushed to hub.docker.com automatically using github actions (workflows). This automation is configured in `.github/workflows/docker-publish.yml`:

- The `Dockerfile.<image_label>_<image_tag>` file is built into the image `<image_label>:<image_tag>`. For example, `Dockerfile.fajnlamp_7.3` is built into the image `fajnlamp:7.3`.
- A file whose tag ends with `.test` is not built.
- A file that has the `.amd64` suffix after the tag is built only for `amd64`. This suffix is not counted as part of the tag, e.g., the file `Dockerfile.fajnlamp_7.3.amd64` is built into the image `fajnlamp:7.3`.

On push to `main`, only the Dockerfiles changed by the push are built (comment/whitespace-only changes are skipped). Images can also be (re)built manually in *Actions*:

- **Build and Push Docker Images** → *Run workflow*: without options it builds the Dockerfiles changed by the last commit; with *Rebuild all Dockerfiles* checked it builds all of them.
- **Build and Push Selected Docker Images** (`.github/workflows/docker-publish-selected.yml`) → *Run workflow*: builds only the Dockerfiles listed in the input field, separated by spaces or commas. Each item may be:
  - a file name with or without the `images/Dockerfile.` prefix, e.g. `fajnlamp_8.3.amd64`,
  - an `<image_label>:<image_tag>` reference (the `.amd64` suffix may be omitted), e.g. `fajnlamp:8.3` or `dev-odoo:19.0-20260926`,
  - a glob pattern, e.g. `prod-odoo_*` or `dev-odoo_19.0-*`.

  An item that matches no file fails the run. `.test` variants are skipped even when selected. From the CLI: `gh workflow run docker-publish-selected.yml -f dockerfiles="fajnlamp:8.3 prod-odoo_*"`.

## Inital GitHub repository configuration

For the workflow to log in to Docker Hub, the following must be configured in the GitHub repository under *Settings → Secrets and variables → Actions*:

| Name                 | Type                             | Value                                                  |
|----------------------|----------------------------------|--------------------------------------------------------|
| `DOCKERHUB_USERNAME` | **Variable** (tab *Variables*)   | Docker Hub username, e.g. `developmentrunsk`           |
| `DOCKERHUB_TOKEN`    | **Secret** (tab *Secrets*)       | Docker Hub Personal Access Token (not the password)    |

- `DOCKERHUB_USERNAME` is read as a variable (`vars.DOCKERHUB_USERNAME`), not as a secret. If it is stored under *Secrets*, it resolves to an empty value and the login fails.
- The token is created on Docker Hub in *Account settings → Personal access tokens → Generate new token* with the **Read & Write** access permission (push and pull).
- Images are always pushed to the `developmentrunsk/` namespace (hardcoded in the workflow), so the account must have push access to it. If `developmentrunsk` is a Docker Hub organization, either use a member account with write access and its PAT, or an Organization Access Token with the organization name as `DOCKERHUB_USERNAME`.
- Alternatively, both values can be set with the `gh` CLI (the secret value is prompted interactively, so it does not end up in the shell history):

    ```bash
    gh variable set DOCKERHUB_USERNAME --body developmentrunsk
    gh secret set DOCKERHUB_TOKEN
    ```

No other configuration is needed - `GITHUB_TOKEN` and the GitHub Actions build cache (`type=gha`) are available automatically.
