# Ansible Monitoring Stack — Node Exporter + Prometheus

Deploy **Node Exporter** and **Prometheus** to any number of Proxmox VMs with a single command.

## Project Structure

```
ansible-monitoring/
├── ansible.cfg
├── site.yml                         # Master playbook
├── inventory/
│   └── hosts.ini                    # Add your VM IPs here
├── group_vars/
│   └── all.yml                      # Versions, ports, paths
└── roles/
    ├── node_exporter/
    │   ├── tasks/main.yml
    │   ├── templates/node_exporter.service.j2
    │   └── handlers/main.yml
    └── prometheus/
        ├── tasks/main.yml
        ├── templates/
        │   ├── prometheus.yml.j2    # Auto-populated from inventory
        │   └── prometheus.service.j2
        └── handlers/main.yml
```

## Quick Start

### 1. Update your inventory
Edit `inventory/hosts.ini` and add your VM IPs:
```ini
[prometheus_server]
prometheus-vm ansible_host=192.168.1.10

[node_exporter_targets]
vm-01 ansible_host=192.168.1.11
vm-02 ansible_host=192.168.1.12
# ... add more
```

### 2. Test connectivity
```bash
ansible all -m ping
```

### 3. Run the playbook
```bash
# Full deploy
ansible-playbook site.yml

# Dry run first
ansible-playbook site.yml --check

# Only node_exporter
ansible-playbook site.yml --tags node_exporter

# Only prometheus
ansible-playbook site.yml --tags prometheus

# Target specific hosts
ansible-playbook site.yml --limit vm-01,vm-02
```

## Customization

Edit `group_vars/all.yml` to change versions or ports:
```yaml
node_exporter_version: "1.8.1"
node_exporter_port: 9100
prometheus_version: "2.52.0"
prometheus_port: 9090
prometheus_retention_time: "15d"
```

## Accessing the Stack

| Service        | URL                              |
|----------------|----------------------------------|
| Prometheus UI  | http://<prometheus_vm_ip>:9090   |
| Node Exporter  | http://<any_vm_ip>:9100/metrics  |

## Adding More VMs Later

Just add the new VM IP to `[node_exporter_targets]` in `hosts.ini` and re-run:
```bash
ansible-playbook site.yml
```
The `prometheus.yml` scrape config will automatically include the new host.
