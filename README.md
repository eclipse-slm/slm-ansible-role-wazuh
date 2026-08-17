# Wazuh Ansible Role

An Ansible role for installing and removing Wazuh backends and agents.

The role supports two service types:

- `backend`: deploys the Wazuh single-node Docker Compose stack.
- `agent`: installs the Wazuh agent package from the Wazuh APT repository.

The service type and desired state are required on every invocation.

## Requirements

Ansible 2.21 or newer is recommended. The role uses the following collections:

- `community.docker`
- `kubernetes.core` (used by the Molecule KubeVirt test setup)
- `ansible.posix`

Install the declared dependencies with:

```bash
ansible-galaxy install -r requirements.yml
```

Backend hosts require Docker Engine and the Docker Compose v2 plugin. The agent installation currently targets Debian-based hosts with `apt`, `systemd`, and `deb822_repository` support.

## Role Variables

### Required variables

| Variable | Allowed values | Description |
| --- | --- | --- |
| `fabos_wazuh_service` | `backend`, `agent` | Selects the Wazuh component to manage. |
| `fabos_wazuh_service_state` | `present`, `absent` | Installs or removes the selected component. |

The role fails early if either variable contains an unsupported value.

### Agent variables

| Variable | Default | Description |
| --- | --- | --- |
| `fabos_wazuh_manager_ip` | No default | Wazuh manager address. Required when installing an agent. |
| `fabos_wazuh_manager_port` | `1514` | Manager listening port passed to the agent package installer. |
| `fabos_wazuh_agent_labels` | `{}` | Mapping of label keys to values written to `ossec.conf`. |
| `fabos_wazuh_default_agent_config_dir` | `/var/ossec/etc` | Directory containing the agent configuration. |

### Backend variables

| Variable | Default | Description |
| --- | --- | --- |
| `fabos_wazuh_backend_version` | `v4.14.6` | Wazuh Docker repository tag to deploy. |
| `fabos_wazuh_compose_project_name` | `wazuh` | Docker Compose project name. |
| `fabos_wazuh_default_compose_ports` | `[]` | Optional list of port mappings with `container_port` and `exposed_port` keys. |
| `fabos_wazuh_backend_monitord_keep_log_days` | `1` | Number of days of manager logs retained by `monitord`. |
| `fabos_wazuh_default_dashboard_base_url` | `""` | Optional Wazuh dashboard base path. |
| `fabos_wazuh_default_dashboard_tls_enabled` | `false` | Whether dashboard TLS should remain enabled. |

The backend uses these default locations:

- Downloaded Wazuh Docker repository: `/tmp/wazuh`
- Compose files: `/opt/wazuh/docker-compose`
- Wazuh configuration: `/opt/wazuh/config`

Additional defaults are defined in [`defaults/main.yml`](defaults/main.yml).

## Examples

### Install a backend

```yaml
---
- name: Install Wazuh backend
  hosts: wazuh_backend
  become: true
  vars:
    fabos_wazuh_service: backend
    fabos_wazuh_service_state: present
    fabos_wazuh_backend_version: v4.14.6
    fabos_wazuh_compose_project_name: wazuh
  roles:
    - fabos.wazuh
```

To expose the dashboard on a host port other than `5601`, configure a mapping such as:

```yaml
fabos_wazuh_default_compose_ports:
  - container_port: 5601
    exposed_port: 5443
```

### Install an agent

```yaml
---
- name: Install Wazuh agent
  hosts: wazuh_agents
  become: true
  vars:
    fabos_wazuh_service: agent
    fabos_wazuh_service_state: present
    fabos_wazuh_manager_ip: 192.0.2.10
    fabos_wazuh_manager_port: 1514
    fabos_wazuh_agent_labels:
      environment: test
      owner: security
  roles:
    - fabos.wazuh
```

### Remove a service

Set `fabos_wazuh_service_state` to `absent` while keeping the appropriate service type:

```yaml
fabos_wazuh_service: agent
fabos_wazuh_service_state: absent
```

Removing a backend stops and removes the Compose workloads and removes the role-managed Compose directory. Removing an agent removes the `wazuh-agent` package.

## Testing

The repository uses Molecule. Activate the project test environment and install dependencies first:

```bash
source ~/.venv/molecule/bin/activate
ansible-galaxy install -r requirements.yml
```

Run the complete default scenario with:

```bash
molecule test
```

Available scenarios include:

- `default`
- `install-wazuh-backend`
- `install-wazuh-agent-ubuntu-24`
- `install-wazuh-full`
- `uninstall-wazuh-agent`
- `uninstall-wazuh-backend`

To run one scenario, use `molecule -s <scenario> test`. For a quick syntax check, use:

```bash
molecule -s <scenario> syntax
```

## Dependencies

The role does not declare Galaxy role dependencies in `meta/main.yml`. The development and Molecule setup downloads `fabos.molecule_kubevirt` and `eclipse-slm.docker` from the roles listed in [`requirements.yml`](requirements.yml).

## License

The role metadata currently contains a placeholder license value. Confirm the intended project license before publishing the role.

## Author

Benjamin Goetz, Fraunhofer IPA
