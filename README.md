# Ansible CD

A lightweight, self-contained continuous-delivery setup for deploying containerized applications to a remote host via Ansible and GitHub Actions.

## What's included

- `ansible_deploy.yml` — a self-contained Ansible playbook that deploys a Docker container to a target host.
- `deploy_workflow.yml` — a GitHub Actions workflow that triggers the Ansible deployment after an upstream workflow completes.

## How it works

1. A GitHub Actions workflow runs after your chosen upstream workflow completes on the `main` branch.
2. The workflow installs Ansible, the `community.docker` collection, and required secrets/vars.
3. Ansible connects to the production host over SSH and runs `ansible_deploy.yml`.
4. The playbook ensures Docker is running, logs into the container registry, pulls the latest image, removes the old container, and starts the updated one.
5. Optional post-tasks (migrations, collectstatic) can be triggered via `RUN_MIGRATIONS` and `COLLECTSTATIC` variables.

## Required GitHub secrets

| Secret | Purpose |
|--------|---------|
| `SERVER_HOST` | Target server IP or hostname |
| `SERVER_USER` | SSH user on the target server |
| `SERVER_SSH_KEY` | Private SSH key for connecting to the server |
| `REGISTRY_USERNAME` | Container registry username |
| `REGISTRY_PASSWORD` | Container registry password or token |
| `IMAGE_NAME` | Full image name, e.g. `myorg/myapp` |

## Required GitHub variables

| Variable | Purpose |
|----------|---------|
| `CONTAINER_NAME` | Name of the running container |
| `CONTAINER_PORTS` | Comma-separated port mappings, e.g. `8000:8000,443:443` |
| `CONTAINER_RESTART_POLICY` | Docker restart policy, e.g. `unless-stopped` |
| `CONTAINER_ENV_JSON` | JSON object of container environment variables |
| `CONTAINER_NETWORKS_JSON` | JSON array of container networks |
| `CONTAINER_NETWORK_MODE` | Docker network mode, e.g. `bridge` |
| `IMAGE_TAG` | Image tag to deploy, e.g. `latest` or commit SHA |

## Optional GitHub variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `RUN_MIGRATIONS` | `false` | Set to `true` to run post-deployment migrations |
| `COLLECTSTATIC` | `false` | Set to `true` to run post-deployment collectstatic |

## Usage

1. Copy `deploy_workflow.yml` into `.github/workflows/` in your repository.
2. Copy `ansible_deploy.yml` into your `ansible/` directory (or update `PLAYBOOKS_DIR` in the workflow).
3. Configure the required secrets and variables in your GitHub repository settings.
4. Update the `workflows` trigger in `deploy_workflow.yml` to match the `name:` of your upstream workflow.
5. Push to `main` and watch the deployment run.

## Notes

- The playbook uses `become: false` by default; adjust if your target host requires privilege escalation.
- `ANSIBLE_HOST_KEY_CHECKING` is disabled for CI convenience. Consider enabling it for stronger security in production.
