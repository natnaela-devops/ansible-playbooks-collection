# Ansible Infrastructure Lab

[![Validate](https://github.com/natnaela-devops/ansible-playbooks-collection/actions/workflows/validate.yml/badge.svg)](https://github.com/natnaela-devops/ansible-playbooks-collection/actions/workflows/validate.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A sanitized, reproducible Ansible lab for preparing Ubuntu hosts with a common operating-system baseline, Docker Engine, and the host prerequisites commonly required by RKE2.

The project is intentionally focused: it demonstrates disciplined infrastructure automation, reusable role design, safe defaults, validation, and idempotency without pretending to be a production cluster installer.

> **Portfolio scope:** This repository is lab work. It is not copied from an employer environment and does not claim that these exact playbooks were deployed in production. Example addresses use documentation-only networks and all environment-specific values must be reviewed before use.

## Architecture

```text
                   GitHub Actions
                        |
             lint / syntax / idempotency
                        |
                        v
+----------------+   Ansible   +-----------------------+
| Control node   | ----------> | Ubuntu target hosts   |
| / CI runner    |             |                       |
+----------------+             |  common baseline      |
                               |  Docker Engine        |
                               |  RKE2 prerequisites   |
                               +-----------------------+
```

More detail is available in [`docs/architecture.md`](docs/architecture.md).

## What this demonstrates

- Role-based Ansible structure with fully qualified collection names
- Reusable defaults instead of hard-coded users and packages
- Docker's signed APT repository using a dedicated keyring
- RKE2 host preparation: swap handling, kernel modules, sysctl settings, and storage prerequisites
- SSH host-key verification enabled by default
- YAML lint, Ansible lint, playbook syntax validation, and repeat-run idempotency checks
- CI automation through GitHub Actions
- Local pre-commit validation support
- Explicit handling of secrets and production-safety boundaries
- No credentials, tokens, kubeconfigs, customer addresses, or production inventory

## Validation coverage

| Check | Purpose |
|---|---|
| `yamllint` | YAML formatting and consistency |
| `ansible-lint` | Ansible best-practice validation |
| `ansible-playbook --syntax-check` | Playbook parsing and syntax |
| second-run idempotency check | Fails if the tested role reports changes on a repeat run |
| pre-commit hooks | Runs lightweight checks before changes are committed |

The current CI idempotency gate covers the safe `common` role on an ephemeral GitHub runner. Docker and RKE2 host preparation are linted and syntax-checked, but they intentionally require a more realistic system environment for full convergence testing.

## Repository layout

```text
.
├── .github/
│   ├── dependabot.yml
│   └── workflows/
│       └── validate.yml
├── docs/
│   ├── architecture.md
│   └── security.md
├── playbooks/
│   ├── common-setup.yml
│   ├── kubernetes-node-prep.yml
│   ├── setup-docker.yml
│   └── site.yml
├── roles/
│   ├── common/
│   ├── docker_engine/
│   └── rke2_node_prep/
├── tests/
│   └── inventory.ci.ini
├── .ansible-lint
├── .pre-commit-config.yaml
├── .yamllint.yml
├── ansible.cfg
├── inventory.ini
├── requirements-ci.txt
└── requirements.yml
```

## Lab inventory

`inventory.ini` uses the TEST-NET-1 range reserved for documentation. Replace the example addresses and users with hosts you control.

```ini
[rke2_servers]
rke2-server-1 ansible_host=192.0.2.10 ansible_user=ubuntu

[rke2_agents]
rke2-agent-1 ansible_host=192.0.2.11 ansible_user=ubuntu

[docker_hosts:children]
rke2_servers
rke2_agents

[rke2_nodes:children]
rke2_servers
rke2_agents
```

## Run the lab

Install required collections first:

```bash
ansible-galaxy collection install -r requirements.yml
```

Review changes before applying them:

```bash
ansible-playbook playbooks/site.yml --check --diff
```

Apply the complete baseline:

```bash
ansible-playbook playbooks/site.yml --ask-become-pass
```

Run one stage only:

```bash
ansible-playbook playbooks/common-setup.yml --ask-become-pass
ansible-playbook playbooks/setup-docker.yml --ask-become-pass
ansible-playbook playbooks/kubernetes-node-prep.yml --ask-become-pass
```

## Validate locally

```bash
python3 -m pip install -r requirements-ci.txt
ansible-galaxy collection install -r requirements.yml
yamllint .
ansible-lint
ansible-playbook --syntax-check playbooks/site.yml
```

Optional pre-commit setup:

```bash
python3 -m pip install pre-commit
pre-commit install
pre-commit run --all-files
```

## Production considerations

This repository deliberately stops short of claiming production readiness. Before adapting it to a real environment, consider at minimum:

- version pinning and upgrade testing for Ansible, Docker, Ubuntu, and RKE2
- separate inventories and variables per environment
- encrypted secret handling through an approved vault or Ansible Vault workflow
- tested backup, recovery, rollback, and maintenance procedures
- stronger integration tests for Docker and RKE2 host preparation
- change review, CI protection, and controlled deployment processes

## Important operating notes

- Disabling swap and changing kernel/sysctl settings affect the host. Use a disposable VM first.
- Adding a user to the `docker` group grants root-equivalent access. The default `docker_engine_users` list is intentionally empty.
- This repository prepares hosts; it does not install RKE2, create a cluster, or prove production readiness.
- Keep secrets in an approved vault or encrypted variable workflow, never in Git.

## Security

See [`docs/security.md`](docs/security.md) for secret-handling guidance and repository safety expectations.

## License

MIT
