# Ansible Role: NUT

[![Build Status](https://github.com/salvoxia/ansible-role-nut/workflows/Release/badge.svg)](https://github.com/Salvoxia/ansible-role-nut/actions/workflows/release.yml)
[![Ansible Galaxy Downloads](https://img.shields.io/badge/dynamic/json?color=blueviolet&label=Galaxy%20Downloads&query=%24.download_count&url=https%3A%2F%2Fgalaxy.ansible.com%2Fapi%2Fv1%2Froles%2F39518%2F%3Fformat%3Djson)](https://galaxy.ansible.com/ui/standalone/roles/salvoxia/nut/)

Installs and configures [NUT](http://networkupstools.org/) (Nework UPS
tools) on Debian based systems, while allowing for advanced NUT user configuration.  
Also supports installing NUT from source tags to gain access to more up-to-date versions if the system's package manager does not provide them.

__Key Features:__
  - Set `nut_state` to either install or remove NUT
  - Allow for advanced NUT user configuration with detailed permission management
  - Install NUT from a specific source tag instead of package manager in case the package manager provided version is too old
  - Switch between specific versions installed from source
  - Switch between installation from package manager and source (or vice versa)

__Dependencies:__
  - [ansible.utils](https://galaxy.ansible.com/ui/repo/published/ansible/utils/)

Install dependencies with
```shell
ansible-galaxy install -r requirements.yml
```
## Installing from Source

By default the role will install NUT using the package manager. If the system's package manager comes with an older NUT package, it is possible to install NUT from source. The role will automatically install all build dependencies, check out the desired source version, compile and install NUT from source.  
It is also possible to use this role for updating NUT installed from source to a newer version (or downgrade to an older version). It is not a 'real' upgrade, but the old version is uninstalled before the new version is installed.
The role supports switching between NUT installed by package manager and installed from source (and vice versa).

The following variables control installation from source:
<table>
  <tr>
    <th>Variable</th>
    <th>Description</th>
  </tr>
  <tr>
    <td> 
      
`nut_install_from_source`
    </td>
    <td>
Flag indicating whether to install NUT from source or not.<br>All the variables below have no effect if not set to `true`.<br>Default: `false`
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_source_repository`
    </td>
    <td>
The URL to the Git Repository to compile NUT from.<br>Default: `https://github.com/networkupstools/nut.git`
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_source_tag`
    </td>
    <td>
The Git Tag to check out before compiling NUT.<br>Can be a branch name as well.<br>Default: `v2.8.2`
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_drivers`
    </td>
    <td>
A list of NUT driver names to compile NUT. <br>For a list fo valid drivers refer to the `Drivers` section in the [generic manual for unified NUT drivers](https://networkupstools.org/docs/man/nutupsdrv.html)<br>Example:
```yaml
nut_drivers:
  - snmp-ups
  - netxml-ups
```
  </td>
  </tr>
</table>

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

<table>
  <tr>
    <th>Attribute</th>
    <th>Description</th>
  </tr>
  <tr>
    <td> 
      
`name`
    </td>
    <td>

Name for the NUT user<br>__Mandatory__
    </td>
  </tr>
  <tr>
    <td> 
      
`password`
    </td>
    <td>
The user's password<br>__Mandatory__
    </td>
  </tr>
  <tr>
    <td> 
      
`role`
    </td>
    <td>
    
Either the `user type` filed of the MONITOR directive in [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html) and/or the `upsmon` directive in [`upsd.users`](https://networkupstools.org/docs/man/upsd.users.html).<br>Allowed vlaues are `primary` and `secondary`<br>__Mandatory__
    </td>
  </tr>
  <tr>
    <td> 
      
`actions`
    </td>
    <td>
    
Used to set allowed actions for the user in [`upsd.users`](https://networkupstools.org/docs/man/upsd.users.html)<br>Only effective effect if `nut_mode` is either `netserver` or `standalone`<br>__Optional__
    </td>
  </tr>
  <tr>
    <td> 
      
`instcmds`
    </td>
    <td>
    
Used to set allowed instant commands for the user in [`upsd.users`](https://networkupstools.org/docs/man/upsd.users.html)<br>Only effective effect if `nut_mode` is either `netserver` or `standalone`<br>__Optional__
    </td>
  </tr>
</table>


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
Explanation for all possible attributes:

<table>
  <tr>
    <th>Attribute</th>
    <th>Description</th>
  </tr>
  <tr>
    <td> 
      
`name`
    </td>
    <td>

An arbitrary string that must identify univocally the UPS<br>__Mandatory__
    </td>
  </tr>
  <tr>
    <td> 
      
`monitoruser`
    </td>
    <td>
The NUT user to use for monitoring this UPS. Must reference an existing user defined in [`nut_users`](#nut_users).<br>__Mandatory__
    </td>
  </tr>
  <tr>
    <td> 
      
`driver`
    </td>
    <td>
    
Depends on your hardware and must be one of the [available NUT driver](http://networkupstools.org/stable-hcl.html). Be sure the NUT version installed on your server has that specific driver available<br>__Mandatory__ for `nut_mode` set to `netserver` or `standalone`
    </td>
  </tr>
  <tr>
    <td> 
      
`device`
    </td>
    <td>
    
Port where the UPS is listening (typically an USB port or a serial device). It is translated to the `port` directive in [`ups.conf`](https://networkupstools.org/docs/man/ups.conf.html)<br>__Mandatory__ for `nut_mode` set to `netserver` or `standalone`
    </td>
  </tr>
  <tr>
    <td> 
      
`description`
    </td>
    <td>
    
An arbitrary string used for debugging and reporting purposes<br>__Optional__
    </td>
  </tr>
  <tr>
    <td> 
      
`monitorhost`
    </td>
    <td>
    
The host that UPS can be monitored with. Is used as the host in the `MONITOR` directive in [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html).<br>__Optional__, default: `localhost`
    </td>
  </tr>
  <tr>
    <td> 
      
`powervalue`
    </td>
    <td>
    
The number of power supplies that the UPS feeds on this system. Is used as `powervalue` in the `MONITOR` directive in [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html).<br>__Optional__, default: `1`
    </td>
  </tr>
  <tr>
    <td> 
      
`powervalue`
    </td>
    <td>
    
`extra` | Optional multiline text to be inserted verbatim in the global section of the relevant configuration file<br>__Optional__
    </td>
  </tr>
</table>


### Misc Variables

All these variables are optional:

<table>
  <tr>
    <th>Variable</th>
    <th>Description</th>
  </tr>
  <tr>
    <td> 
      
`nut_enable_service`
    </td>
    <td>

Flag indicating whether to start the services appropriate for the installed `nut_packages` after configuration.<br>Default: `true`
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_upssched_cmdscript_content`
    </td>
    <td>

Contents of a custom `upssched-cmd` script.
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_upssched_cmdscript`
    </td>
    <td>

Path on the Ansible host to a custom `upssched-cmd` script. Sets the `CMDSCRIPT` directive in [`upssched.conf`](https://networkupstools.org/docs/man/upssched.conf.html).<br>If `nut_upssched_cmdscript_content` is not empty, the contents of that variable will be stored under this path.
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_upssched_pipefn`
    </td>
    <td>

Sets the `PIPEFN` directive in [`upssched.conf`](https://networkupstools.org/docs/man/upssched.conf.html).<br>Default: `/run/nut/upssched/upssched.pipe`
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_upssched_lockfn`
    </td>
    <td>

Sets the `LOCKFN` directive in [`upssched.conf`](https://networkupstools.org/docs/man/upssched.conf.html).<br>Default: `/run/nut/upssched/upssched.lock`
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_upssched_at_commands`
    </td>
    <td>

Define `AT` directives for [`upssched.conf`](https://networkupstools.org/docs/man/upssched.conf.html).<br>Example:
```yaml
nut_upssched_at_commands:
  - notifytype: ONBATT
    upsname: *
    command: START-TIMER commbad 30
```
   </td>
  </tr>
  <tr>
    <td> 
      
`nut_upsmon_notify`
    </td>
    <td>

Define custom `NOTIFYMSG` and `NOTIFYFLAG` directives for [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html).<br>Example:
```yaml
nut_upsmon_notify:
  - name: ONLINE
    msg: UPS %s on line power
    flag: SYSLOG+WALL+EXEC
  - name: ONBATT
    msg: UPS %s on battery
    flag: SYSLOG+WALL+EXEC
```
   </td>
  </tr>
  <tr>
    <td> 
      
`nut_upsmon_notifycmd_content`
    </td>
    <td>

Contents of a custom upsmon notification script.
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_upsmon_notifycmd`
    </td>
    <td>

Path to a custom notification script. Sets the `NOTIFYCMD` directive in [`upsmon.conf`](https://networkupstools.org/docs/man/upsmon.conf.html).<br>If `nut_upsmon_notifycmd_content` is not empty, the contents of that variable will be stored under that path.
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_ups_extra`
    </td>
    <td>

Additional content to append to the `ups.conf` file
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_upsd_extra`
    </td>
    <td>

Additional content to append to the `upsd.conf` file
    </td>
  </tr>
  <tr>
    <td> 
      
`nut_upsmon_extra`
    </td>
    <td>

Additional content to append to the `upsmon.conf` file
    </td>
  </tr>
</table>


## Example Playbook

```yaml
- hosts: all
  roles:
  - role: salvoxia.nut
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

For more examples, please see `molecule/*/converge.yml` and `molecule/*/prepare.yml`.


## Cheat Sheet

Run a Docker container with systemd:
```bash
sudo docker run --tmpfs /tmp --tmpfs /run -v /sys/fs/cgroup:/sys/fs/cgroup:rw --cgroupns=host --privileged --name sysd --rm geerlingguy/docker-debian11-ansible
```
## License
MIT

## Author Information

This role was created in 2016 by Nicola Fontana (ntd@entidi.it).  
This fork is modified and maintained by Salvoxia (salvoxia@blindfish.info).