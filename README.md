Ansible Role: NUT
=================
[![Build Status](https://github.com/simoncaron/ansible-role-nut/workflows/Release/badge.svg)](https://github.com/simoncaron/ansible-role-nut/actions?query=workflow%3Release)
[![Ansible Galaxy Downloads](https://img.shields.io/badge/dynamic/json?color=blueviolet&label=Galaxy%20Downloads&query=%24.download_count&url=https%3A%2F%2Fgalaxy.ansible.com%2Fapi%2Fv1%2Froles%2F32010%2F%3Fformat%3Djson)](https://galaxy.ansible.com/ui/standalone/roles/simoncaron/nut/)

Installs and configures [NUT](http://networkupstools.org/) (Nework UPS
tools) on Debian based systems.

## Role Variables

Available variables are listed below, along with default values (see
`defaults/main.yml`).

### `nut_mode`
```yaml
nut_mode: standalone
```

The NUT `MODE` setting in [`nut.conf`](https://networkupstools.org/docs/man/nut.conf.html).  
Allowed values in this role are `standalone`, `netserver` or `netclient`.

### `nut_managed_config`
```yaml
nut_managed_config: true
```

If this is set to false, none of the following options will have any
effect, that is any and all changes under `/etc/nut/` will be your
responsibility. This is often desirable when you have complex
configurations.

### `nut_users`

This role allows for configuring different NUT users with specific `actions` and `instcmds`. These only have effect if `nut_mode` is set to either `netserver` or `standalone`.  
The minimal user configuration looks like this:
```yaml
nut_users:
  - name: monitor
    password: Whatever
    role: secondary
```

A full user configuration might look like this:
```yaml
nut_users:
  - name: monitor
    password: Changeme
    role: secondary
  - name: admin
    password: uNro<FQ.eav:fyc^.,>,b29/4Dts=9
    role: primary
    actions:
      - fsd
      - set
    instcmds:
      - all
```

Explanation for all possible attributes:
| Attribute   |  Description   |
| :------------------- | :------------ |
| `name`      | Name for the NUT user<br>__Mandatory__ |
| `password` | The user's password<br>__Mandatory__ |
| `role` | Either the `user type` filed of the MONITOR directive in [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html) and/or the `upsmon` directive in [`upsd.users`](https://networkupstools.org/docs/man/upsd.users.html). Allowe vlaues are `primary` and `secondary`<br>__Mandatory__ |
| `actions` | Used to set allowed actions for the user in [`upsd.users`](https://networkupstools.org/docs/man/upsd.users.html)<br>Only effective effect if `nut_mode` is either `netserver` or `standalone`<br>__Optional__ |
| `instcmds` | Used to set allowed instant commands for the user in [`upsd.users`](https://networkupstools.org/docs/man/upsd.users.html)<br>Only effective effect if `nut_mode` is either `netserver` or `standalone`<br>__Optional__ |


### `nut_ups`

The `nut_ups` variable is a list of UPS definitions the NUT installation knows of. Depening on `nut_mode`, the minimal set of attributes differs.

For `nut_mode = netclient`, the minimal UPS configuration looks like this:
```yaml
nut_ups:
  - name: UPS
    monitoruser: monitor
``` 

For `nut_mode = netserver` or `nut_mode = standalone`, the minimal UPS configuration looks like this:
```yaml
nut_ups:
  - name: UPS
    monitoruser: monitor
    driver: usbhid-ups
    device: auto
``` 

The full configuration for `nut_ups` with all possible attributes looks like this:
```yaml
nut_ups:
  - name: UPS
    monitoruser: monitor
    driver: usbhid-ups
    device: auto
    description: Some descriptive information
    monitorhost: localhost
    powervalue: 1
    extra: |
      maxretry = 10
      retrydelay = 1
```
Explanation for all possible attributes
| Attribute   |  Description   |
| :------------------- | :------------ |
| `name`      | An arbitrary string that must identify univocally the UPS<br>__Mandatory__ |
| `monitoruser` | The NUT user to use for monitoring this UPS. Must reference an existing user defined in [`nut_users`](#nut_users).<br>__Mandatory__ |
| `driver` | Depends on your hardware and must be one of the [available NUT driver](http://networkupstools.org/stable-hcl.html). Be sure the NUT version installed on your server has that specific driver available<br>__Mandatory__ for `nut_mode` set to `netserver` or `standalone`|
| `device` | Port where the UPS is listening (typically an USB port or a serial device). It is translated to the `port` directive in [`ups.conf`](https://networkupstools.org/docs/man/ups.conf.html)<br>__Mandatory__ for `nut_mode` set to `netserver` or `standalone`|
| `description` | An arbitrary string used for debugging and reporting purposes<br>__Optional__ |
| `monitorhost` | The host that UPS can be monitored with. Is used as the host in the `MONITOR` directive in [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html).<br>__Optional__, default: `localhost` |
| `powervalue` | The number of power supplies that the UPS feeds on this system. Is used as `powervalue` in the `MONITOR` directive in [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html).<br>__Optional__, default: `1` |
| `extra` | Optional multiline text to be inserted verbatim in the global section of the relevant configuration file<br>__Optional__ |


### Misc Variables

All these variables are optional:
| Variable   |  Description   |
| :------------------- | :------------ |
| `nut_services`      | List of service names to enable<br>Default: `nut-driver-enumerator`, `nut-monitor`, `nut-server` |
| `nut_enable_service` | Flag indicating whether to start the services defined in `nut_services` after configuration. Default: `true` |
| `nut_upssched_cmdscript_content` | Contents of a custom `upssched-cmd` script. |
| `nut_upssched_cmdscript` | Path on the Ansible host to a custom `upssched-cmd` script. Sets the `CMDSCRIPT` directive in [`upssched.conf`](https://networkupstools.org/docs/man/upssched.conf.html).<br>If `nut_upssched_cmdscript_content` is not empty, the contents of that variable will be stored under that path. |
| `nut_upssched_pipefn` | Sets the `PIPEFN` directive in [`upssched.conf`](https://networkupstools.org/docs/man/upssched.conf.html).<br>Default: `/run/nut/upssched/upssched.pipe`|
| `nut_upssched_lockfn` | Sets the `LOCKFN` directive in [`upssched.conf`](https://networkupstools.org/docs/man/upssched.conf.html).<br>Default: `/run/nut/upssched/upssched.lock`|
| `nut_upssched_at_commands` | Define `AT` directives for [`upssched.conf`](https://networkupstools.org/docs/man/upssched.conf.html).<br>Refer to [defaults/main.yml](defaults/main.yml) for an example. |
| `nut_upsmon_notify` | Define custom `NOTIFYMSG` and `NOTIFYFLAG` directives for [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html).<br>Refer to [defaults/main.yml](defaults/main.yml) for an example. |
| `nut_upsmon_notifycmd` | Path to a custom notification script. Sets the `NOTIFYCMD` directive in [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html). |
| `nut_upsmon_notifycmd_content` | Contents of a custom upsmon notification script. |
| `nut_upsmon_notifycmd` | Path to a custom notification script. Sets the `NOTIFYCMD` directive in [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html).<br>If `nut_upsmon_notifycmd_content` is not empty, the contents of that variable will be stored under that path. |
| `nut_ups_extra` | Additional content to append to the `ups.conf` file |
| `nut_upsd_extra` | Additional content to append to the `upsd.conf` file |
| `nut_upsmon_extra` | Additional content to append to the `upsmon.conf` file |

## Example Playbook

```yaml
- hosts: all
  roles:
  - role: ntd.nut
    nut_users:
      - name: monitor
        password: changeme
        role: secondary
    nut_ups:
      - name: riello
        monitoruser: monitor
        driver: riello_usb
        device: /dev/ups
        description: iPlug 800
```

For more examples, please see `tests/test.yml`.

## License
MIT

## Author Information

This role was created in 2016 by Nicola Fontana (ntd@entidi.it).  
This fork is modified and maintained by Salvoxia (salvoxia@blindfish.info).