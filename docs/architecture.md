# Architecture

## Purpose

This lab separates host preparation into small Ansible roles so each concern can be reviewed, reused, and tested independently.

```text
                    +----------------------+
                    | GitHub Actions       |
                    | lint / syntax / CI   |
                    +----------+-----------+
                               |
                               v
+-------------------+    SSH / Ansible    +---------------------------+
| Control node      | ------------------> | Ubuntu target hosts       |
| or CI runner      |                     |                           |
+-------------------+                     | common baseline           |
                                          | Docker Engine            |
                                          | RKE2 prerequisites       |
                                          +---------------------------+
```

## Roles

### `common`

Applies a reusable base package set to supported Debian-family systems.

### `docker_engine`

Configures Docker's signed Ubuntu APT repository, installs Docker Engine and plugins, starts the Docker service, and optionally grants approved users access to the `docker` group.

### `rke2_node_prep`

Prepares a host for RKE2 by managing swap, required kernel modules, Kubernetes networking sysctls, storage prerequisites, and iSCSI service state.

## Execution flow

`playbooks/site.yml` composes the individual stages. Operators can run the full baseline or execute each stage independently during testing and troubleshooting.

## Design principles

- **Idempotency:** repeat runs should converge without unnecessary changes.
- **Least surprise:** potentially privileged settings are explicit and documented.
- **Separation of concerns:** roles have narrowly defined responsibilities.
- **Safe examples:** inventories contain documentation-only addresses and no credentials.
- **Validation before execution:** linting and syntax checks run automatically in CI.
- **Honest scope:** host preparation is intentionally separate from cluster installation and production lifecycle management.

## Production extension path

A real deployment would normally add environment-specific inventories, encrypted secret handling, version pinning, stronger integration testing, controlled RKE2 server/agent installation, upgrade procedures, and post-deployment validation.
