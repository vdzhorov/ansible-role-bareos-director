# Ansible role Bareos director

Ansible role for installing and configuring Bareos director on a target host.

## What Bareos functionality you can configure

* Clients
* Pools
* Volumes
* Schedules
* Filesets
* Remote/Local storages

## Getting Started

Get all other roles related to Bareos. Keep in mind that funckionality has been split into different roles for overall better control. The other roles are:

* vdzhorov.ansible_role_bareos_base
* vdzhorov.ansible_role_bareos_client
* vdzhorov.ansible_role_bareos_storage
* vdzhorov.ansible_role_bareos_webui [Optional]

### Example inventory

```
[bareos_director]
director1 ansible_host=1.2.3.4 diraddress=localhost dirport=9101

[bareos_director:vars]
role=director

[bareos_storage]
remote-storage1 ansible_host=2.2.3.4

[bareos_storage:vars]
role=storage

[bareos_clients]
client1.example.com ansible_host=3.2.3.4

[bareos_clients:vars]
role=client
```

### Example playbook

```
---

- hosts: bareos_director
  roles:
    - 'ansible-role-bareos-director'
    - 'ansible-role-bareos-webui'

- hosts: bareos_storage
  roles:
    - 'ansible-role-bareos-storage'

- hosts: bareos_clients
  roles:
    - 'ansible-role-bareos-client'
```

### Example vars

#### Group_vars all.yml

```
---

bareos_version: '19.2'

bareos_clients_password: 'verysecurepassword'
bareos_storage_password: 'anotherverysecurepassword'

bareos_owner: bareos
bareos_group: bareos

bareos_remote_storage_locations:
  - name: remote_storage1
    address: "1.2.3.4"
    device: '/var/lib/bareos/storage'
```

#### Group_vars bareos_director.yml

```
---

# Webui
bareos_webui: True
bareos_webui_username: 'admin'
bareos_webui_password: 'adminsecurepassword'

# Client config
bareos_clients:
  - hostname: 'client1.example.com'
    address: '1.2.3.4'
    schedule: 'Weekly'
    fileset: 'LinuxAll'
    pool:
      retention: '7 days'
      use_duration: '6d'
      storage: 'remote_storage1'

# Schedules
bareos_schedules:
  - name: "Weekly"
    cycle: |
      Run = Full sat at 00:00
      Run = Incremental mon-fri at 00:00

bareos_filesets:
  - name: "LinuxAll"
    description: "Backup all regular filesystems, determined by filesystem type."
    options:
      backed_up_directories:
        - "/"
      include: |
        Signature = MD5 # calculate md5 checksum per file
        One FS = No     # change into other filessytems
        FS Type = btrfs
        FS Type = ext2  # filesystems of given types will be backed up
        FS Type = ext3  # others will be ignored
        FS Type = ext4
        FS Type = reiserfs
        FS Type = jfs
        FS Type = xfs
        FS Type = zfs
      exclude: |
        File = /var/lib/bareos/storage
        File = /proc
        File = /tmp
        File = /var/tmp
        File = /var/log/journal
        File = /.journal
        File = /.fsck
  - name: "LinuxAllwithMySQLdump"
    description: "Backup all filesystem except /var/lib/mysql. This is handled by a plugin"
    options:
      backed_up_directories:
        - "/"
        - "Plugin = 'python:module_path=/usr/lib64/bareos/plugins:module_name=bareos-fd-mysql'"
      include: |
        Signature = MD5 # calculate md5 checksum per file
        One FS = No     # change into other filessytems
        FS Type = btrfs
        FS Type = ext2  # filesystems of given types will be backed up
        FS Type = ext3  # others will be ignored
        FS Type = ext4
        FS Type = reiserfs
        FS Type = jfs
        FS Type = xfs
        FS Type = zfs
      exclude: |
        File = /var/lib/bareos/storage
        File = /proc
        File = /tmp
        File = /var/tmp
        File = /var/log/journal
        File = /.journal
        File = /.fsck
        File = /var/lib/mysql

```

#### Group_vars bareos_storage.yml

```
---

bareos_max_concurrent_jobs: 5
bareos_max_bandwidth_per_job: "50 m/s"

bareos_archive_type: "File"
bareos_archive_device: "/var/lib/bareos/storage"

```

### Example ansible run command

`ansible-playbook -i inventories/bareos/production playbooks/bareos.yml`

## Built With

 [Ansible 2.8](https://docs.ansible.com/ansible/2.8/index.html) - Ansible 2.8

## Built With

* [Ansible 2.8](https://docs.ansible.com/ansible/latest/roadmap/ROADMAP_2_8.html) - Ansible 2.8

## Authors

* **Valentin Dzhorov** - *Initial work* - [vdzhorov](https://github.com/vdzhorov)

## Company
* **Delta.BG**

## License

This project is licensed under the MIT License.
