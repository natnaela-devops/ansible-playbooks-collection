# Ansible Infrastructure Lab

[![Validate](https://github.com/natnaela-devops/ansible-playbooks-collection/actions/workflows/validate.yml/badge.svg)](https://github.com/natnaela-devops/ansible-playbooks-collection/actions/workflows/validate.yml)

A sanitized, reproducible lab for preparing Ubuntu hosts with a common operating-system baseline, Docker Engine, and the host prerequisites used by RKE2. The repository demonstrates Ansible structure, idempotent task design, safe defaults, linting, syntax validation, and a repeat-run idempotency check.

This is portfolio and lab work. It is not copied from an employer environment and does not claim that these exact playbooks were deployed in production. Addresses use documentation-only networks, and every environment-specific value must be reviewed before use.

## What this demonstrates

- Role-based Ansible structure with fully qualified collection names
- Reusable defaults instead of hard-coded users and packages
- Docker's signed APT repository using a dedicated keyring
- RKE2 host preparation: swap handling, kernel modules, sysctl settings, and storage packages
- SSH host-key verification enabled by default
- YAML lint, Ansible lint, playbook syntax checks, and an automated second-run idempotency gate
- No credentials, tokens, kubeconfigs, customer addresses, or production inventory

## Repository layout

```text
.
├── .github/workflows/validate.yml
├── playbooks/
│   ├── common-setup.yml
│   ├── kubernetes-node-prep.yml
│   ├── setup-docker.yml
│   └── site.yml
├── roles/
│   ├── common/
│   ├── docker_engine/
│   └── rke2_node_prep/
├── tests/inventory.ci.ini
├── ansible.cfg
├── inventory.ini
└── requirements-ci.txt
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
yamllint .
ansible-lint
ansible-playbook --syntax-check playbooks/site.yml
```

The GitHub Actions workflow also converges the safe `common` role twice on an ephemeral runner and fails if the second run reports a change.

## Important operating notes

- Disabling swap and changing kernel/sysctl settings affect the host. Use a disposable VM first.
- Adding a user to the `docker` group grants root-equivalent access. The default `docker_engine_users` list is intentionally empty.
- This repository prepares hosts; it does not install RKE2, create a cluster, or prove production readiness.
- Pin and test Ansible, Docker, Ubuntu, and RKE2 versions for each real environment.
- Keep secrets in an approved vault or encrypted variable workflow, never in Git.

## License

MIT
