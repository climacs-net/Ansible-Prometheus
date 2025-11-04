# Quick Guide Steps for macOS and GitHub CI

## Running the Playbook on macOS

1. **Install prerequisites**
   - Install [Homebrew](https://brew.sh/) if it is not already available on your system.
   - Use Homebrew to install Ansible:
     ```bash
     brew install ansible
     ```
   - Confirm that Python 3 is available (it ships with recent macOS versions and is required by Ansible).

2. **Prepare SSH access**
   - Ensure the SSH private key expected by the project is present at `~/.ssh/id_ed25519`.
   - The default remote user is `ec2-user` (as configured in `ansible.cfg`). Confirm this user has sudo privileges on the target host.

3. **Update inventory information**
   - The provided inventory targets the `prometheus` host group with `prometheus.climacs.net`.
   - Replace this hostname with your own target host(s) if needed before running the playbook.

4. **Run the playbook**
   - Execute the playbook from the repository root:
     ```bash
     ansible-playbook playbook.yml
     ```
   - If you want to be explicit about the inventory file, run:
     ```bash
     ansible-playbook -i hosts playbook.yml
     ```
   - The play requires privilege escalation (`become: yes`). Make sure your SSH key grants sudo access on the target machine.

## Running Checks in GitHub Actions

Add a GitHub Actions workflow (for example, `.github/workflows/ci.yml`) similar to the following to perform syntax checks in CI without touching production systems:

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  ansible-checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install Ansible
        run: pip install ansible
      - name: Syntax check
        run: ansible-playbook playbook.yml --syntax-check
      # Optional: add ansible-lint if introduced later
```

### Secrets and Integration Runs

- Workflows that run the playbook against real infrastructure must inject secrets such as SSH keys and inventory data via GitHub Secrets (e.g., `ANSIBLE_PRIVATE_KEY`).
- Limit CI to `--syntax-check` or disposable test hosts unless appropriate secrets and safeguards are in place, because the playbook installs packages, creates users, and manages services on target systems.
