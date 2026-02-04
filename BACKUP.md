# Backups

Backups are done via a [systemd timer](https://github.com/status-im/infra-role-systemd-timer) and [`pg_dump`](https://www.postgresql.org/docs/current/app-pgdump.html):
```
admin@node-01.do-ams3.sites.aptly:~ % sudo systemctl list-timers '*-aptly-*'
NEXT                        LEFT    LAST                        PASSED  UNIT                      ACTIVATES                  
Thu 2026-02-05 00:00:00 UTC 9h left Wed 2026-02-04 00:00:04 UTC 14h ago backup-aptly-server.timer backup-aptly-server.service
```
The backup saves the whole `/data/aptly-server` directory excluding `downloads` temporary dir:
```
jakubgs@node-01.do-ams3.sites.aptly:~ % r
ID        Time                 Host                         Tags        Paths               Size
------------------------------------------------------------------------------------------------------
bec83e3e  2026-02-04 00:00:05  node-01.do-ams3.sites.aptly  aptly       /data/aptly-server  13.264 GiB
------------------------------------------------------------------------------------------------------
1 snapshots
```

# Restoring

## Backup Existing Data

Stop the API service and copy Aptly data folder:
```bash
sudo systemctl stop aptly-api.service
cd /data
sudo cp -r aptly-server bpk_aptly-server
```

## Restore Backup

To restore backup to the same directory the destination permissions need to be changed:
```bash
sudo chmod o+w -R /data/aptly-server
sudo -i -u restic restic restore --target=/ 12345678
sudo chmod o-w -R /data/aptly-server
```
```
Summary: Restored 1105 files/dirs (6.633 GiB) in 1:56
```
By using `--target=/` we restore the backup to the same location from which it was copied.

## Checking Repo State

```
 > sudo -u aptly aptly snapshot list | tail -n6
 * [v25.9.0]: status-im/nimbus-eth2 release v25.9.0 on 2025-09-04
 * [v25.9.1]: status-im/nimbus-eth2 release v25.9.1 on 2025-09-26
 * [v25.9.2]: status-im/nimbus-eth2 release v25.9.2 on 2025-09-26
 * [v26.1.0]: status-im/nimbus-eth2 release v26.1.0 on 2026-01-29

To get more information about snapshot, run `aptly snapshot show <name>`.

 > sudo -u aptly aptly snapshot verify v26.1.0
Loading packages...
Verifying...
Missing dependencies (3):
  lsb-release [amd64]
  lsb-release [arm64]
  lsb-release [armhf]
```
