# Ansible Systemd-Unit
## Description
Generate and configure Systemd Unit files both globally and for lingering users.

## General Usage
Simply requires the `systemd_units` dictionary.
Each key under `systemd_units` represents a username, `root` will configure system units in `/etc/systemd/system`.
Under each username, specify if the user should be allowed to linger, defaults to yes.
Then include the unit type: `unit`, `timer`, `mount`, etc...
For each unit type, include the configuration for the unit.

## Unit Definition
The `config` key holds the INI style configuration for the unit file. The keys are case sensitive to the unit file options, this method allows you to define any section and option for the unit file.
Options may be defined as an array to be entered more than one time without overriding each other.  

### ***This role will overwrite existing unit files!***

## Unit Options
The following options are avaible to be set per unit definition:
- enabled: will set the unit to be enabled at boot
- state: any valid state value, defaults to started
- restart_on_change: will restart the unit if we make any modification to the unit definition, defaults to yes
- backup_before_write: will create a single .bak of the unit prior to writing our definition, defaults to yes
- filepath: allows you to override the default unit file path. Defaults to either /etc/systemd/system for root or ~/.config/systemd/user for users

```
---
systemd_units:
  USERNAME: # Username of root will handle system unit in /etc/systemd/system, other users will handle ~/.config/systemd/user
    enable_linger: # Defaults to yes
    UNIT_EXTENSION: # service, target, socket, etc...
      UNIT_NAME: # filename of the unit
        config: # Contains the options that comprise of the unit file
          SECTION:  # Unit, Service, Install, etc...
            OPTION: # Can either be OPTION=value or OPTION=ARRAY for options that can be listed multiple times.
        options: # Options that apply to the unit, but do not go into the unit file
          enabled: # Defaults to yes
          state: # Defaults to started, choices: reloaded;restarted;started;stopped.
          restart_on_change: # Defaults to yes
          backup_before_write: # Defaults to yes
          filepath: Defaults to either /etc/systemd/system for root or ~/.config/systemd/user for users
```

## Example: Logrotate
File: `/usr/lib/systemd/system/logrotate.service`
```
[Unit]
Description=Rotate log files
Documentation=man:logrotate(8) man:logrotate.conf(5)
ConditionACPower=true

[Service]
Type=oneshot
ExecStart=/usr/sbin/logrotate /etc/logrotate.conf

# performance options
Nice=19
IOSchedulingClass=best-effort
IOSchedulingPriority=7

# hardening options
#  details: https://www.freedesktop.org/software/systemd/man/systemd.exec.html
#  no ProtectHome for userdir logs
#  no PrivateNetwork for mail deliviery
#  no ProtectKernelTunables for working SELinux with systemd older than 235
#  no MemoryDenyWriteExecute for gzip on i686
PrivateDevices=true
PrivateTmp=true
ProtectControlGroups=true
ProtectKernelModules=true
ProtectSystem=full
RestrictRealtime=true
```

file: `/usr/lib/systemd/system/logrotate.timer`
```
[Unit]
Description=Daily rotation of log files
Documentation=man:logrotate(8) man:logrotate.conf(5)

[Timer]
OnCalendar=daily
AccuracySec=12h
Persistent=true

[Install]
WantedBy=timers.target
```
Variable to configure: (***NOTE: We do not write comment lines!***)
```
systemd_units:
  root:
    service:
      logrotate:
        config:
          Unit:
            Description: Rotate log files
            Documentation: 'man:logrotate(8) man:logrotate.conf(5)'
            ConditionACPower: 'true'
          Service:
            Type: oneshot
            ExecStart: /usr/sbin/logrotate /etc/logrotate.conf
            Nice: 19
            IOSchedulingClass: best-effort
            IOSchedulingPriority: 7
            PrivateDevices: 'true'
            PrivateTmp: 'true'
            ProtectControlGroups: 'true'
            ProtectKernelModules: 'true'
            ProtectSystem: full
            RestrictRealtime: 'true'
        options:
          enabled: false
          filepath: /usr/lib/systemd/system/logrotate.service
    timer:
      logrotate:
        config:
          Unit:
            Description: Daily rotation of log files
            Documentation: 'man:logrotate(8) man:logrotate.conf(5)'
          Timer:
            OnCalendar: daily
            AccuracySec: 12h
            Persistent: 'true'
          Install:
            WantedBy: timers.target
        options:
          filepath: /usr/lib/systemd/system/logrotate.timer
```


## Support
For support, please raise an issue and provide the following items
- Sample task/playbook to replicate your issue
- Resultant file that is created.
- If modifying an existing file, please provide the unmodified version as well.
