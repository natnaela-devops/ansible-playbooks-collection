# Security and Safety Notes

This repository is intentionally sanitized for public portfolio use.

## Do not commit

- passwords, API keys, access tokens, SSH private keys, or kubeconfigs
- customer, employer, or production IP addresses and hostnames
- private registry credentials
- cloud credentials or service-account keys
- unencrypted vault files

## Secrets

For real environments, use an approved secret-management solution such as Ansible Vault or an external vault and keep the decryption material outside the repository.

Example workflow:

```bash
ansible-vault encrypt group_vars/production/vault.yml
ansible-playbook playbooks/site.yml --ask-vault-pass
```

Do not use the example above as a reason to store plaintext secrets before encryption.

## Docker group warning

Membership in the `docker` group is effectively root-equivalent on a typical Docker host. The role therefore leaves `docker_engine_users` empty by default and requires an explicit operator choice before adding users.

## Host-level changes

The RKE2 preparation role changes kernel modules, sysctl settings, swap configuration, packages, and service state. Test these changes on disposable or non-production hosts before adapting the role to a real environment.

## Public repository hygiene

Before publishing infrastructure code:

1. replace real addresses with documentation ranges such as `192.0.2.0/24`
2. remove organization-specific names and internal DNS records
3. inspect Git history for secrets, not only the current files
4. validate `.gitignore` rules
5. review diffs before every push

## Reporting

If you notice sensitive information in this repository, do not repost it publicly. Contact the repository owner through the GitHub profile so the material can be reviewed and removed safely.
