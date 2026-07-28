# AAP Certificate Demo

This repository contains a small Ansible Automation Platform demo for managing a web service certificate through a full lifecycle: verify access, discover the current certificate, rotate it, validate the deployment, roll back if needed, and reset the environment for another run.

The demo appears to target an NGINX-backed service that serves HTTPS on `web01.lab.test:443` and stores its certificate material under `/etc/pki/cert-demo`.

## What’s in the demo

- `playbooks/00_verify_access.yml` checks that Ansible can reach the managed host, validates NGINX, confirms the health endpoint, and prints certificate details.
- `playbooks/01_discover_certificate.yml` reads the live certificate from NGINX, calculates remaining validity, and records discovery data for later workflow jobs.
- `playbooks/02_rotate_certificate.yml` backs up the current certificate and key, generates a replacement certificate, deploys it safely, and reloads NGINX.
- `playbooks/03_validate_certificate.yml` validates the deployed certificate from the execution environment and can optionally simulate a failure to exercise rollback.
- `playbooks/04_rollback_certificate.yml` restores the previous certificate from backup, reloads NGINX, and verifies the service after rollback.
- `playbooks/99_reset_demo.yml` rebuilds the demo back to an initial short-lived certificate and clears previous backups.

## Requirements

- Ansible or AAP with access to the target host(s).
- A Linux host running NGINX with OpenSSL installed.
- Passwordless privilege escalation for the tasks that manage certificate files and NGINX.
- A certificate deployment path at `/etc/pki/cert-demo/tls.crt` and `/etc/pki/cert-demo/tls.key`.
- A working HTTPS health endpoint at `/healthz`.

## Configuration

The included `ansible.cfg` disables host key checking, disables retry files, and uses the default stdout callback:

- `host_key_checking = False`
- `retry_files_enabled = False`
- `stdout_callback = default`
- `interpreter_python = auto_silent`

Adjust inventory, credentials, and execution environment settings in AAP as needed for your environment.

## Typical workflow

1. Run the access check to confirm the managed host and service are reachable.
2. Discover the current certificate and record whether rotation is required.
3. Rotate the certificate and keep a backup of the previous key pair.
4. Validate the new certificate from the execution environment.
5. If validation fails, run the rollback playbook to restore the previous certificate.
6. Use the reset playbook to return the demo to its initial state before another demonstration.

## Running the playbooks

Use your normal inventory and connection settings. For example:

```bash
ansible-playbook -i <inventory> playbooks/00_verify_access.yml
ansible-playbook -i <inventory> playbooks/01_discover_certificate.yml
ansible-playbook -i <inventory> playbooks/02_rotate_certificate.yml
ansible-playbook -i <inventory> playbooks/03_validate_certificate.yml
ansible-playbook -i <inventory> playbooks/04_rollback_certificate.yml
ansible-playbook -i <inventory> playbooks/99_reset_demo.yml
```

To exercise the rollback path intentionally, run the validation playbook with `simulate_validation_failure=true`.

## Notes

- The discovery and validation playbooks preserve useful values with `set_stats` so they can be reused by AAP workflow jobs.
- The rotation and rollback playbooks create and consume timestamped backups under `/var/backups/cert-demo`.
- The demo uses OpenSSL to inspect and generate certificates, so the target host needs the relevant command-line tools installed.