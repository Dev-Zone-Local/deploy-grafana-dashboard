# Grafana Dashboards Ansible Project

This workspace contains playbooks and an Ansible role to import and manage Grafana dashboards from JSON files. You can either use the role-driven deployment or a processing playbook that normalizes datasource placeholders before importing.

## Overview
- Role: [roles/grafana_dashboards](roles/grafana_dashboards)
  - Dashboard JSONs: [roles/grafana_dashboards/files/dashboards](roles/grafana_dashboards/files/dashboards)
  - Defaults: [roles/grafana_dashboards/defaults/main.yml](roles/grafana_dashboards/defaults/main.yml)
  - Tasks: [roles/grafana_dashboards/tasks/main.yml](roles/grafana_dashboards/tasks/main.yml)
- Playbooks:
  - Role-based deploy: [deploy_dashboards.yml](deploy_dashboards.yml)
  - Process + import: [import_dashboards.yml](import_dashboards.yml)
- Configuration:
  - Global vars: [group_vars/all.yml](group_vars/all.yml)
  - Collections: [collections/requirements.yml](collections/requirements.yml)

## Prerequisites
- Ansible 2.12+
- Grafana reachable from your machine and a valid API key with write permissions
- Required collection(s):
  - `community.grafana` (declared in [collections/requirements.yml](collections/requirements.yml))

Install collections:

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

## Configuration
Set basic connection and auth settings in [group_vars/all.yml](group_vars/all.yml):

```yaml
grafana_url: "http://localhost:3000"
grafana_api_key: "<your_api_key>"
grafana_overwrite: true
grafana_validate_certs: false
```

You can also review defaults in [roles/grafana_dashboards/defaults/main.yml](roles/grafana_dashboards/defaults/main.yml). Key variables:
- `grafana_url`: Grafana base URL
- `grafana_api_key`: API key (required)
- `grafana_overwrite`: Allow updates to existing dashboards
- `grafana_validate_certs`: TLS cert validation toggle
- `grafana_commit_message`: Commit message used on import
- `dashboards_dir`: Directory to scan for dashboard JSON files
- `dashboards`: Optional list of dashboard files to import; defaults to all JSONs under role `files/dashboards`
- `datasource_name`: Fallback datasource name for placeholder replacement
- `grafana_tmp_dir`: Temp directory for processed dashboards

## Usage
### Option A: Role-based deploy
Runs the role to import dashboards. The role will:
- Auto-discover JSON files under [roles/grafana_dashboards/files/dashboards](roles/grafana_dashboards/files/dashboards) unless `dashboards` is provided
- Replace common datasource placeholders (e.g., `${DS_PROMETHEUS}`, `$datasource`) using the detected Prometheus datasource name/UID
- Write processed dashboards to a temp directory, then import them via `community.grafana.grafana_dashboard`

Run:

```bash
ansible-playbook -i inventory deploy_dashboards.yml
```

Override variables inline if needed:

```bash
ansible-playbook -i inventory deploy_dashboards.yml \
  -e grafana_url="http://grafana.example:3000" \
  -e grafana_api_key="<key>"
```

Target specific files:

```bash
ansible-playbook -i inventory deploy_dashboards.yml \
  -e dashboards='["roles/grafana_dashboards/files/dashboards/7645_rev289.json","roles/grafana_dashboards/files/dashboards/4475_rev5.json"]'
```

### Option B: Process + import playbook
[import_dashboards.yml](import_dashboards.yml) pre-processes dashboard JSONs to replace common datasource placeholders (e.g., `${DS_PROMETHEUS}`, `$datasource`) with the detected Prometheus datasource name/UID, writes the processed files to `/tmp`, then imports them via `community.grafana.grafana_dashboard`.

Run:

```bash
ansible-playbook -i inventory import_dashboards.yml \
  -e grafana_url="http://localhost:3000" \
  -e grafana_api_key="<key>"
```

## Datasource Resolution
- The role and the import playbook query Grafana datasources and pick the first Prometheus datasource.
- If no Prometheus datasource is returned, the replacement falls back to `datasource_name`.

## Troubleshooting
- 401/403 errors: Verify `grafana_api_key` and its permissions.
- TLS issues: Set `grafana_validate_certs: false` for self-signed setups (development only).
- Datasource not found in import playbook: The playbook falls back to provided `datasource_name`; set it explicitly via `-e datasource_name=Prometheus` if detection fails.

## Project Structure
- Inventory: [inventory](inventory) (plays target `localhost` by default)
- Dashboards: place JSON files under [roles/grafana_dashboards/files/dashboards](roles/grafana_dashboards/files/dashboards)

## License
No license specified.
