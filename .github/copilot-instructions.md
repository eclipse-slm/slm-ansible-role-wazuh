# Wazuh Ansible Role - Copilot Instructions

## Build, Test, and Lint Commands

This role uses **Molecule** for testing with Kubernetes/KubeVirt.

### System Requirements

- **Ansible**: 2.21.1 (installed via apt)
- **Molecule**: 26.6.0 with Docker and Kubevirt drivers (installed via pip)
- **Molecule virtual environment**: `~/.venv/molecule`

### Running Tests

```bash
# Activate virtual environment first
source ~/.venv/molecule/bin/activate

# Run all tests
molecule test

# Converge (apply role)
molecule converge

# Verify
molecule verify
```

### Scenarios

- **Default scenario**: `molecule -s default`
- **Install Wazuh backend**: `molecule -s install-wazuh-backend`

### Syntax Check

```bash
ansible-playbook -i tests/inventory tests/test.yml --syntax-check
```

### Key Molecule Commands

```bash
# Test sequence runs: dependency -> cleanup -> destroy -> syntax -> create -> prepare -> converge -> idempotence -> verify -> cleanup -> destroy

# Install dependencies first
molecule dependency

# Create test instance
molecule create

# Run role on test instance
molecule converge

# Verify state
molecule verify
```

### Docker & Kubernetes MCP

This project includes MCP server configurations for Docker and Kubernetes at `~/.copilot/mcp-servers.json`:

- **Docker**: Container management, build, run, inspect
- **Docker Compose**: Compose file management
- **Kubernetes**: Cluster access and resource management

### Installation

**Ansible & Molecule setup:**

```bash
# Install system packages
sudo apt-get install -y ansible python3.12-venv python3-pip

# Create virtual environment
python3 -m venv ~/.venv/molecule
source ~/.venv/molecule/bin/activate

# Install Molecule and drivers
pip install molecule molecule-docker molecule-kubevirt==0.1.0

# Install Ansible Galaxy dependencies
ansible-galaxy install -r requirements.yml
```

## High-Level Architecture

This is an **Ansible Galaxy role** that installs Wazuh (a security monitoring platform) in two modes:

### Role Structure

- **`tasks/main.yml`**: Entry point that validates inputs and routes to `backend.yml` or `agent.yml`
- **`tasks/backend.yml`**: Handles Wazuh backend installation via Docker Compose
- **`tasks/agent.yml`**: Placeholder for Wazuh agent installation (not yet implemented)
- **`defaults/main.yml`**: Default variables including:
  - `fabos_wazuh_default_version`: Wazuh version (default: `v4.14.6`)
  - `fabos_wazuh_default_backend_compose_url`: Remote Docker Compose template URL
  - Service types: `backend`, `agent`
  - Service states: `present`, `absent`

### Backend Installation Flow

1. Downloads Wazuh's official Docker Compose from GitHub (versioned via tag)
2. Creates `/opt/wazuh` directory
3. Starts Wazuh stack with `community.docker.docker_compose_v2`
4. Includes `eclipse-slm.docker` in prepare phase (Molecule)

### Test Infrastructure

- **Molecule default**: Uses KubeVirt to create Kubernetes test instances
- **Molecule install-wazuh-backend**: Tests backend installation on Ubuntu 24.04 container
- Test inventory: `localhost` (connection: local)

## Key Conventions

### Variable Naming

- **Prefix**: All role variables use `fabos_wazuh_` prefix
- **Defaults**: `fabos_wazuh_default_*` for default values
- **Required**: `fabos_wazuh_service` and `fabos_wazuh_service_state` must be set via vars (empty by default)

### Validation

The role validates inputs in `tasks/main.yml` using `ansible.builtin.assert`:
- `fabos_wazuh_service` must be one of `fabos_wazuh_default_service_types`
- `fabos_wazuh_service_state` must be one of `fabos_wazuh_default_service_state_types`

### Docker Compose

- Template: `templates/docker-compose-backend.yml.j2`
- Configurable via variables like `fabos_wazuh_manager_version`, `fabos_wazuh_api_version`
- Volumes: `wazuh-data`, `wazuh-logs`, `wazuh-ssl`
- Default ports: 514/UDP (manager), 55000 (manager API), 443 (API)

### Development Setup

- **Dev container**: `.devcontainer/devcontainer.json` configured
- **Gitignore**: Includes `local.env` for sensitive data

### Ansible Best Practices

- **Force handlers**: `--force-handlers` flag used in Molecule
- **No host key checking**: Enabled in Molecule config
- **Pipelining**: Enabled for SSH performance
- **Color output**: `ANSIBLE_FORCE_COLOR=1` set in Molecule