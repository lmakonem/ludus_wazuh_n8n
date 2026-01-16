# Wazuh + n8n Lab (Ludus)

Author: **Howard Mukanda** (@ITSecurityLabs on YouTube and X)

This Ludus role deploys a **single-node Wazuh stack (Docker)** and an
**n8n + Postgres automation server (Docker)** on a Debian/Ubuntu Linux
VM. It is designed for blue-team / purple-team labs where students query
Wazuh alerts via n8n workflows.
<img width="1351" height="719" alt="image" src="https://github.com/user-attachments/assets/4e35d382-6d1a-4622-affb-a1fda4bbb531" />

<img width="2554" height="1320" alt="image" src="https://github.com/user-attachments/assets/09137f1a-63fa-4e2a-a57b-07faa80f6a8c" />
------------------------------------------------------------------------

## 🧱 What this role does

When run on a Linux VM in Ludus, this role:

-   Installs Docker Engine and the Docker Compose plugin (if needed).
-   Sets `vm.max_map_count=262144` for Wazuh indexer stability.
-   Deploys **n8n** with:
    -   A dedicated **Postgres 16** container.
    -   Persistent volumes for Postgres and `/home/node/.n8n`.
    -   `N8N_SECURE_COOKIE=false` so learners can use
        `http://VM_IP:5678` without TLS in the lab.
-   Deploys **Wazuh 4.9.0 single-node** via the official `wazuh-docker`
    repo:
    -   Runs `generate-indexer-certs.yml` to create indexer certs.
    -   Starts manager, indexer, and dashboard containers.
-   Connects the **n8n** container to the **Wazuh single-node Docker
    network** so n8n can query the Wazuh indexer over HTTPS on port
    9200.

------------------------------------------------------------------------

## 🔧 Prerequisites

### On your Ludus host

-   Ludus is installed and working.
-   Default Linux templates (e.g. `debian-12-x64-server-template`) are
    built.
-   You have SSH / shell access to the Ludus host to add roles.

### On the target VM

-   Must be a **Linux VM** (Debian or Ubuntu), created and managed by
    Ludus.

------------------------------------------------------------------------

## 📦 Adding the role to Ludus

``` bash
ludus ansible roles add https://github.com/<your-username>/ludus_wazuh_n8n.git
```

### Confirm it shows up

``` bash
ludus ansible roles list
```

You should see `ludus_wazuh_n8n` in the list.

------------------------------------------------------------------------

## 🧩 Example Ludus range config

``` yaml
ludus:
  - vm_name: "{{ range_id }}-wazuh-n8n-server"
    hostname: "{{ range_id }}-wazuh-n8n-server"
    template: debian-12-x64-server-template
    vlan: 20
    ip_last_octet: 10
    ram_gb: 8
    cpus: 4
    linux: true
    testing:
      snapshot: false
      block_internet: false
    roles:
      - ludus_wazuh_n8n
    role_vars:
      wazuh_repo_url: "https://github.com/wazuh/wazuh-docker.git"
      wazuh_version: "v4.9.0"

      n8n_postgres_user: "n8n"
      n8n_postgres_password: "changemeStrong123"
      n8n_postgres_db: "n8n"

      n8n_port: 5678
      n8n_timezone: "America/Chicago"
      n8n_secure_cookie: false
```

------------------------------------------------------------------------

## 🚀 Deployment

``` bash
ludus range config get > config.yml
nano config.yml
ludus range config set -f config.yml
ludus range deploy -t user-defined-roles
```

------------------------------------------------------------------------

## 🔍 Accessing the lab

-   n8n: `http://<VM_IP>:5678`
-   Wazuh: `https://<VM_IP>` (admin / SecretPassword)

------------------------------------------------------------------------

## 🔗 Using n8n to query Wazuh

-   Base URL: `https://single-node-wazuh.indexer-1:9200`
-   Index: `wazuh-alerts-*`
-   Ignore SSL issues (self-signed)

------------------------------------------------------------------------

## 🧪 Next steps

-   Query alerts
-   Automate responses
-   Extend with Wazuh agents
