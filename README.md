# Description

This role configures [KeyDB](https://docs.keydb.dev/) running in a Docker container.

* It supports single instance, replica, and multi-master topologies. 
* Auth is enforced with the default user + password (`key_db_password`) — used by both clients and replication. 
* No TLS is configured (assume private network).
* Includes host-level tuning: disables Transparent Huge Pages (THP), sets `vm.overcommit_memory=1`.

# Configuration

These settings are all mandatory:
```yaml
aptly_server_repo_domain: 'apt.example.org'
aptly_server_service_name: 'aptly-server'
# GitHub
aptly_server_github_token:    '{{ lookup("vault", "gh", field="token") }}'
# GPG
aptly_server_gpg_key_id:      '{{ lookup("vault", "gpg", field="id") }}'
aptly_server_gpg_private_key: '{{ lookup("vault", "gpg", field="priv") }}'
aptly_server_gpg_public_key:  '{{ lookup("vault", "gpg", field="pub") }}'
aptly_server_gpg_password:    '{{ lookup("vault", "gpg", field="pass") }}'
# Repositories
aptly_server_repositories:
  - name: 'example'
    distro: 'all'
    component: 'main'
```

# Management

The `aptly-api` service runs as a System service:
```
jakubgs@node-01.do-ams3.sites.aptly:~ % sudo systemctl -o cat status aptly-api.service    
● aptly-api.service - Aptly REST API service
     Loaded: loaded (/etc/systemd/system/aptly-api.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2025-07-23 07:20:28 UTC; 2 months 24 days ago
   Main PID: 2668398 (aptly)
      Tasks: 7 (limit: 2309)
     Memory: 13.6M
        CPU: 11min 21.920s
     CGroup: /system.slice/aptly-api.service
             ├─2668398 /usr/bin/aptly api serve -listen=localhost:8080
             └─2904358 gpg-agent --homedir /home/aptly/.gnupg --use-standard-socket --daemon

Loading packages...
Generating metadata files and linking package files...
Finalizing metadata files...
Signing file 'Release' with gpg, please enter your passphrase when prompted:
Clearsigning file 'Release' with gpg, please enter your passphrase when prompted:
[GIN] 2025/09/26 - 21:14:52 | 200 |  4.003849714s |       127.0.0.1 | PUT      "/api/publish/nimbus/all"
Cleaning up prefix "nimbus" components main...
```
The API is used by the `publish.sh` script to create snapshots and publish them.

# Backup & Restore

See [BACKUP.md](./BACKUP.md) doc.
