# ansible-role-bacula

This Role can install any component of the backup system "bacula".

## quick bacula introduction

| Component | function | variable (ansible-tag) |
| ------------- |:-------------:| -----:|
| bacula-director (dir) | central management, coordination on other components | bacula_dir_role (bacula-dir) |
| bacula-storage daemon (sd) | controls the writing device(s) | bacula_sd_role (bacula-sd) |
| bacula-console | management console, connects to bacula-dir | bacula_console_role (bacula-console) |
| bacula-file daemon (fd) | agent on target system | bacula_fd_role (bacula-fd) |

see the notes on firewalling: https://www.bacula.org/9.6.x-manuals/en/problems/Dealing_with_Firewalls.html

## Requirements
- installed database-server (mysql or pgsql) on director

## Limitations

Currently only tested on mysql / mariadb and Ubuntu/Debian in LTS-Versions.

## Quick start

### customize (at least) this vars:
(see example playbooks).

bacula_dir_fqdn: "fqdn.of.director.domain.tld"
bacula_sd_fqdn: "fqdn.of.sd.domain.tld"

bacula_dir_restore_path: "/srv/bacula-restores" (Default restore path)

-> DB engine (can be 'pgsql' or 'mysql')
bacula_dir_db_engine: mysql

### Define the director/SD via (host_vars / group_vars):

host_vars/bacula-dir1.stefanux.net.yml

```yaml
---
bacula_console_role: True
bacula_dir_role: True
bacula_sd_role: True
bacula_fd_role: True
```

## Variables

### defaults

defaults are set here:
- defaults/main.yml
- vars/{{ ansible_os_family }}.yml

### vaulted passwords
my recommendation: set the following variables in your vault:

```yaml
bacula_console_password: ""
bacula_dir_password: ""
bacula_dir_dbpassword: ""
bacula_mon_password: ""
bacula_sd_password: ""
```


why?
empty passwords are substituted with random values - on very run.
Except for the fd-passwords which will be preseved when bacula_fd_auto_psk is True (default).

### customize storage device (tape, Filestorage etc.)
Example: Ringbuffer 
- oldest data is recycled/overwritten when the storage is full
- size is bacula_sd_pool_max_volumes x bacula_sd_pool_max_volume_byte
Example: 10G x 100 Volumes means maximum capacity is ~1TB backupspace, depending on the amount

### Storage Definitions
bacula_sd_device_name: "RingBuffer"
bacula_sd_media_type: "File"

**where to store the volumes**:

bacula_sd_archive_device: "/srv/bacula"
bacula_dir_pool_name: "Files"
bacula_dir_pool_label_format: "Bacula-Vol-"
bacula_dir_pool_recycle: "yes"

bacula_dir_pool_file_retention: "180 days" # Default: 60 days

bacula_dir_pool_job_retention: "24 months" # Default: 6 months

**Prune expired Jobs/Files**:

bacula_dir_pool_autoprune: "yes"
bacula_dir_pool_volume_rention: "3650 days"
bacula_dir_pool_max_volume_bytes: "10G"
bacula_dir_pool_max_volumes: 100

### Host overrides

Example: host_vars/host1.domain.tld.yml

**own fileset**:
```yaml
bacula_fd_fileSet_name: "fileset-of-host1"
bacula_fd_fileSet_includes: |+
File = "/files/only/on/host1"
File = "/etc"
File = "/root"
bacula_fd_fileSet_excludes: |+
File = "/files/EXCLUDED/on/host1"
File = "/proc"
```

**additional options in job-ressource**:
```yaml
bacula_fd_extra_job_options: |+
Option1 = Example1
Option2 = Example2
```

same goes **for client and jobdef**-ressources:

bacula_fd_extra_client_options
bacula_fd_extra_jobdefs_options

**different FQDN for connected**:
```yaml
bacula_fd_connect_address: "different.fqdn.to.connect.to"
```

### Change Schedules:

- see "bacula-dir-fileset-default.conf.j2" for a example.
- define your own template with additional Schedule: "bacula_dir_schedules_extra_emplate"
and reference them in "bacula_fd_schedule"
