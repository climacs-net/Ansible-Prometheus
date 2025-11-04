# Project Summary: Ansible-Prometheus

## Overview
Ansible-Prometheus provides an Ansible playbook for provisioning a Prometheus monitoring server on an Amazon Linux host. The automation downloads the Prometheus release archive, installs the binaries, configures Prometheus, sets up a dedicated system user, and manages the Prometheus service with systemd.

## Repository Structure
- `playbook.yml` – Main Ansible playbook that updates packages, installs dependencies, retrieves Prometheus v2.41.0, configures directories and files, and manages the Prometheus systemd service.
- `ansible.cfg` – Sets the default inventory to `hosts`, specifies the SSH key location, and defines the default remote user (`ec2-user`).
- `hosts` – Static inventory listing a single host in the `prometheus` group (`prometheus.climacs.net`).
- `README.md` – Placeholder README file lacking setup instructions.
- `playbook.yml.old` – Previous version of the playbook that uses the `copy` module for deploying binaries.

## Operational Flow
1. Update all YUM packages on the target host.
2. Install prerequisite packages (`wget`, `tar`).
3. Download and extract the Prometheus v2.41.0 Linux AMD64 release archive.
4. Move Prometheus binaries (`prometheus`, `promtool`) into `/usr/local/bin` and apply executable permissions.
5. Create `/etc/prometheus` with a minimal `prometheus.yml` configuration and install a systemd service definition.
6. Create a `prometheus` system user, set up the data directory at `/var/lib/prometheus`, reload systemd, and enable/start the service.

## Notable Configuration Choices
- Assumes availability of an SSH private key at `~/.ssh/id_ed25519` for connecting as `ec2-user`.
- Deploys Prometheus version 2.41.0 explicitly; upgrades require manual updates to the URL and versioned paths.
- The systemd service runs Prometheus under the dedicated `prometheus` user and uses default console assets under `/usr/share/prometheus`.

## Recommendations
1. **Enhance the README:** Document prerequisites, usage instructions, and variables to improve onboarding.
2. **Parameterize Versioning:** Introduce variables for the Prometheus version and download URL to simplify upgrades.
3. **Use Idempotent File Tasks:** Replace `command: mv` tasks with `copy` or `install` modules (as seen in `playbook.yml.old`) to ensure idempotency and avoid repeated downloads.
4. **Add Handlers:** Convert service reload and restart steps into handlers triggered by configuration changes to reduce unnecessary restarts.
5. **Introduce Validation:** Add tasks to verify service status and expose Prometheus availability (e.g., via `uri` module) after deployment.
