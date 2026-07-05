# tcapi Analysis

## Overview

This document analyzes `tcapi`, the command-line configuration interface used by the ASUS DSL-AC750 firmware.

During the investigation, `tcapi` was identified as a key component in the router's configuration management architecture. It provides access to configuration nodes, allows runtime changes, commits configuration updates, and interacts with the central `cfg_manager` daemon.

The analysis was performed directly on the running router through an SSH session.

---

# Role of tcapi

`tcapi` acts as a user-space configuration interface.

It is used to:

- Read configuration values
- Display configuration nodes
- Modify runtime configuration
- Commit configuration changes
- Save configuration persistently
- Communicate with `cfg_manager`

The available commands were discovered by executing:

```bash
tcapi
```

Output:

```text
set
unset
get
show
commit
save
read
readAll
staticGet
```

---

# Command Overview

| Command | Purpose |
|---------|---------|
| set | Modify a configuration value |
| unset | Remove a configuration value |
| get | Read a single configuration value |
| show | Display all values under a node |
| commit | Apply changes for a node |
| save | Save configuration persistently |
| read | Read a configuration node |
| readAll | Read all configuration data |
| staticGet | Read static configuration data |

---

# Configuration Nodes

The firmware stores configuration in named nodes.

Examples discovered during the analysis:

| Node | Purpose |
|------|---------|
| SSH_Entry | SSH / Dropbear configuration |
| Firewall_Entry | Firewall and web access settings |
| Dhcpd_Common | DHCP pool configuration |
| Dproxy_Entry | DNS proxy configuration |
| Syslog_Entry | Logging configuration |
| Timezone_Entry | Time and NTP configuration |
| Misc_Entry | Miscellaneous system settings |
| WLan_Entry0 | Wireless SSID and security configuration |
| WLan_Common | Wireless radio configuration |
| SysInfo_Entry | Device and firmware information |

---

# Example: Reading SSH Configuration

The SSH configuration was inspected using:

```bash
tcapi show SSH_Entry
```

Example output:

```text
Enable=Yes
timeout=20
sshport=22
Need_Pass=1
Authkeys=
```

This configuration corresponds to the running Dropbear SSH process:

```text
dropbear -p 22 -I 1200 -a
```

The value `timeout=20` represents 20 minutes, which matches the Dropbear idle timeout value of 1200 seconds.

---

# Example: DHCP Configuration

The DHCP configuration was inspected using:

```bash
tcapi show Dhcpd_Common
```

Example output:

```text
start=192.168.1.2
pool_count=253
end=192.168.1.254
lease=86400
```

This indicates that the DHCP server assigns addresses from:

```text
192.168.1.2 - 192.168.1.254
```

with a lease time of:

```text
86400 seconds
```

---

# Example: DNS Proxy Configuration

The DNS proxy configuration was inspected using:

```bash
tcapi show Dproxy_Entry
```

Example output:

```text
Active=Yes
type=1
Primary_DNS=8.8.8.8
Secondary_DNS=8.8.4.4
```

This configuration is used by the router to define upstream DNS servers.

---

# Example: Syslog Configuration

The syslog configuration was inspected using:

```bash
tcapi show Syslog_Entry
```

Example output:

```text
remoteSyslogEnable=0
Log_Firewall=1
Log_Config=1
Log_Auth=1
Log_DHCP=1
Log_Other=1
LogEnable=Yes
WriteLevel=7
DisplayLevel=6
remotePort=514
```

A controlled runtime configuration change was tested using:

```bash
tcapi set Syslog_Entry DisplayLevel 7
tcapi show Syslog_Entry
tcapi commit Syslog_Entry
```

The value changed successfully at runtime.

The configuration was then restored:

```bash
tcapi set Syslog_Entry DisplayLevel 6
tcapi commit Syslog_Entry
```

This confirmed that `tcapi set` modifies the runtime configuration database and `tcapi commit` applies the change to the relevant subsystem.

---

# tcapi and libtcapi

The `tcapi` binary itself is small:

```text
/userfs/bin/tcapi
```

During analysis, the related shared library was identified:

```text
/lib/libtcapi.so
/lib/libtcapi.so.1
/lib/libtcapi.so.1.4
```

All three library files had the same MD5 hash, indicating that they are identical copies or versioned aliases of the same library.

The library contains symbols and strings related to configuration operations:

```text
tcapi_set
tcapi_get
tcapi_show
tcapi_commit
tcapi_save
tcapi_read
tcapi_readAll
```

---

# Communication with cfg_manager

Further analysis showed that `tcapi` communicates with `cfg_manager` using a Unix Domain Socket.

The socket path was identified as:

```text
/tmp/tcapi_sock
```

Strings found inside `libtcapi.so` included:

```text
send2CfgManager
socket
connect
send
recv
/tmp/tcapi_sock
```

This indicates that `tcapi` is not directly modifying configuration files. Instead, it sends requests to `cfg_manager`.

---

# Runtime Verification

The `cfg_manager` process was inspected through `/proc`.

Open file descriptors showed:

```text
fd 3 -> socket
```

The socket inode was matched using:

```bash
netstat -an
```

Result:

```text
unix ... LISTENING ... /tmp/tcapi_sock
```

This confirmed that `cfg_manager` listens on `/tmp/tcapi_sock`.

---

# Context Switch Verification

To confirm runtime interaction, context switch counters were observed before and after executing a `tcapi` query.

Before:

```bash
cat /proc/<cfg_manager_pid>/status | grep ctxt
```

Then:

```bash
tcapi get SSH_Entry Enable
```

After running the query, the context switch counters increased.

This confirmed that `tcapi` requests are handled by the running `cfg_manager` process.

---

# Configuration Workflow

The observed configuration workflow is:

```text
tcapi CLI
    │
    ▼
libtcapi.so
    │
    ▼
Unix Domain Socket
/tmp/tcapi_sock
    │
    ▼
cfg_manager
    │
    ▼
Configuration Database
    │
    ▼
Runtime Services
```

---

# set, commit and save

The firmware exposes three important configuration stages:

```text
set
commit
save
```

Their likely roles are:

| Operation | Role |
|----------|------|
| set | Change runtime configuration value |
| commit | Apply configuration changes to services |
| save | Persist configuration to flash storage |

The `save` mechanism appears to generate a romfile configuration and write it to the `romfile` MTD partition.

Evidence found in `cfg_manager` includes:

```text
tcapi_save
/tmp/var/romfile.cfg
/userfs/bin/mtd write %s romfile
```

This suggests the following persistence flow:

```text
tcapi save
    │
    ▼
create romfile
    │
    ▼
/tmp/var/romfile.cfg
    │
    ▼
mtd write
    │
    ▼
romfile partition
```

---

# Key Findings

Key findings from the `tcapi` analysis:

- `tcapi` is the primary configuration CLI.
- Configuration is organized into named nodes.
- `tcapi` supports runtime read, write, commit and save operations.
- `tcapi` communicates with `cfg_manager` through `/tmp/tcapi_sock`.
- `libtcapi.so` implements the communication layer.
- `cfg_manager` acts as the actual configuration server.
- `commit` applies changes to runtime services.
- `save` appears to persist configuration into flash storage.

---

# Commands Used

```bash
tcapi

tcapi show SSH_Entry

tcapi show Dhcpd_Common

tcapi show Dproxy_Entry

tcapi show Syslog_Entry

tcapi set Syslog_Entry DisplayLevel 7

tcapi commit Syslog_Entry

ls -lh /lib/libtcapi*

grep -a "tcapi_" /lib/libtcapi.so.1

grep -a "/tmp/tcapi_sock" /lib/libtcapi.so.1

netstat -an

cat /proc/<cfg_manager_pid>/status
```

---

# Conclusion

The `tcapi` framework is one of the most important components of the ASUS DSL-AC750 firmware.

It provides a structured interface for reading, modifying, applying and saving configuration data. Rather than directly editing configuration files, `tcapi` communicates with the central `cfg_manager` daemon using a Unix Domain Socket.

This architecture separates the user interface, configuration API and service management logic into different layers, making `tcapi` a central part of the firmware's runtime configuration system.
