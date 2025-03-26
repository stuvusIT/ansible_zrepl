# zrepl Ansible Role

This role installs and configures zrepl.

## Requirements

This role should probably work on reasonably up to date Debian/Ubuntu systems, but we only tested it with Debian 12.

## Role Variables

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role.
Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Don't forget to indent the markdown table so it is readable even if not rendered.

| Name           |  Required/Default  | Description                                                                                           |
| -------------- | :----------------: | ----------------------------------------------------------------------------------------------------- |
| `zrepl_config` | :heavy_check_mark: | zrepl configuration. This variable will be serialized to YAML and written to `/etc/zrepl/zrepl.yaml`. |

## Hold Snapshots

This role also places a hook under `/opt/zrepl-hook-hold` that, when used, will add a ZFS hold named `zrepl` to newly created snapshots.
This is useful if you want to then backup these snapshots using some backup tool such as [zfs-restic-uploader](https://github.com/stuvusIT/ansible_zfs_restic_uploader).
In this case the backup tool is responsible for releasing the ZFS hold.
When you configure pruning, zrepl can only prune snapshots on which the ZFS hold has been released.

This role currently doesn't support deploying custom hooks, so if you need a hook that does something else, you need to deploy it via some other mechanism or create a PR on this role.

## Example

```yml
- hosts: hypervisor01
  become: true
  roles:
    - role: zrepl
      zrepl_config:
        global:
          logging:
            - type: stdout
              level: warn
              format: human
        jobs:
          - name: snapjob_s3
            type: snap
            filesystems:
              /tank/dataset1
              /tank/dataset2
            snapshotting:
              type: cron
              cron: "0 2 * * *"
              prefix: zrepl_s3_
              timestamp_format: iso-8601
              hooks:
                - type: command
                  path: /opt/zrepl-hook-hold
                  timeout: 30s
            pruning:
              keep:
                - type: grid
                  grid: 1x30d(keep=all)
                  regex: ^zrepl_s3_.*
                - type: regex
                  negate: true
                  regex: ^zrepl_s3_.*
```

## License

This work is licensed under the [MIT License](./LICENSE).
