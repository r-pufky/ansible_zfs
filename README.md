# ZFS
ZFS Pool and snapshot management for ansible hosts.

## [Requirements][i]
Requires [r_pufky.deb][g] galaxy-ng collection. See
[reference documentation][h] for specific ZFS configuration.

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults][j] - User configurable options.

## Usage
Manages existing ZFS pools, scrubbing schedules, and snapshot backups. Use
`community.general.zfs` or manual commands to manage ZFS pool creation. All
options must be explicitly set.

### Warning
> ZFS kernel module will be automatically loaded and the specified pools
> mounted.
>
> Secure boot requires signed modules; if the MOK is not loaded and secure boot
> is enabled, the module will fail to load with the message:
>
>   /sbin/modprobe zfs
>   > modprobe: ERROR: could not insert 'zfs': Key was rejected by service
>
> This requires installing the MOK (machine owners key) before applying this
> role.

### Feature Flags
Tasks are gated by feature flags and executed in the following order.

  Step | Flag                          | Notes
 ------|-------------------------------|-------
  1    | zfs_flg_kernel_module_restart | Restart machine if modprobe fails.
  2    | zfs_flg_force_pools           | Force import pools.
  3    | zfs_flg_snapshot              | Enable snapshot sync.
  4    | zfs_flg_pools_share_restart   | Export NFS/SMB shares.

### Example Playbooks

``` yaml
# Automatic backup of snapshots to a remote server via zincrsend over SSH.
# See snapshot defaults.
- name: 'Manage ZFS pools, enable monthly scrub, weekly dataset snapshots.'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.zfs'
  vars:
    zfs_flg_snapshot: true
    zfs_srv_pools:
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
    zfs_cfg_snapshot_schedule: 'weekly'
    zfs_cfg_snapshot_retention: 3
    zfs_cfg_snapshot_datasets:
      - 'tank/documents'
      - 'tank/home'
      - 'tank/backup'
      - 'tank/software'
    zfs_cfg_snapshot_remote_pool: 'backup_tank'
    zfs_cfg_snapshot_remote_server: 'example.com'
    zfs_cfg_snapshot_remote_ssh_options:
      - '-i'
      - '/root/backup/backup.key'
```

## Development
Configure [environment][a].

``` bash
# Run all tests.
molecule test --all
```

### [Releases][b]

  Release | Debian | Ansible | Notes
 ---------|--------|---------|-------
  3.x.x   | 13     | 2.20    | Ansible 2.20, feature flags, and semantic versioning.
  2.x.x   | 13     | 2.18    | Migrate to Debian Trixie.
  1.x.x   | 12     | 2.18    | Use standardized libraries.
  0.x.x   | 12     | 2.18    | Migration from private repository.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License][c] | [direct link][f]

## Author Information
PGP: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9][d] | [github gist][e]

[a]: https://r-pufky.github.io/ansible_docs
[b]: https://semver.org/spec/v2.0.0
[c]: https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0
[d]: https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9
[e]: https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22

[f]: https://github.com/r-pufky/ansible_zfs/blob/main/LICENSE
[g]: https://github.com/r-pufky/ansible_collection_deb
[h]: https://r-pufky.github.io/docs/filesystem/zfs/
[i]: https://github.com/r-pufky/ansible_zfs/blob/main/meta/main.yml
[j]: https://github.com/r-pufky/ansible_zfs/tree/main/defaults/main/main.yml