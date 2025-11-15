# ZFS
ZFS Pool and snapshot management for ansible hosts.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_zfs/blob/main/meta/main.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_zfs/blob/main/defaults/main)

## Dependencies
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.deb](https://github.com/r-pufky/ansible_collection_deb) collection.

## Example Playbook
Manages existing ZFS pools, scrubbing schedules, and snapshot backups. Secure
boot enabled systems should apply [r_pufky.deb.secure_boot](https://github.com/r-pufky/ansible_secure_boot)
before applying this role to enable secure boot ZFS support.

Use `community.general.zfs` to manage ZFS pool creation.

The role provides a built in user, however, a consumer provided user is highly
recommended.

Manage a ZFS pool, enabling monthly scrubs, and weekly dataset snapshots with
automatic backup of those snapshots to a remote server via `zincrsend` over
`SSH`.

All options must be explicitly set.

``` yaml
- name: 'Manage ZFS pools'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.zfs'
  vars:
    zfs_pools:
      - name: 'tank'
        scrub: 'monthly'
        datasets:
          - name: 'documents'
            properties:
              - option: 'atime'
                value: 'off'
              - option: 'xattr'
                value: 'sa'
              - option: 'dnodesize'
                value: 'auto'
              - option: 'sharenfs'
                value: 'rw=10.10.10.0/24,sec=sys,mountpoint'
          - name: 'backup'
            properties:
              - option: 'atime'
                value: 'off'
              - option: 'compression'
                value: 'lz4'
    zfs_snapshot_enable: true
    zfs_snapshot_schedule: 'weekly'
    zfs_snapshot_retention: 3
    zfs_snapshot_datasets:
      - 'tank/documents'
      - 'tank/home'
      - 'tank/backup'
      - 'tank/software'
    zfs_snapshot_remote_pool: 'backup_tank'
    zfs_snapshot_remote_server: 'example.com'
    zfs_snapshot_remote_ssh_options:
      - '-i'
      - '/root/backup/backup.key'
```

## Development
Configure [environment](https://r-pufky.github.io/ansible_collection_docs/ansible/environment)

Run all unit tests:
``` bash
molecule test --all
```

### Releases
Major release versions track Debian release versions:

* **[13.x.x](https://github.com/r-pufky/ansible_zfs)**: 13 Trixie.
* **[12.x.x](https://github.com/r-pufky/ansible_zfs/tree/12.x)**: 12 Bookworm.

### Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_zfs/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
