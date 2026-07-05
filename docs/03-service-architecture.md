# Service Architecture Analysis

## Overview

After understanding the boot process, the next objective was identifying the runtime service architecture of the ASUS DSL-AC750.

Rather than operating as a single monolithic application, the firmware is composed of multiple independent daemons, each responsible for a specific subsystem.

The purpose of this document is to analyze these services, their responsibilities, and how they interact during normal router operation.

The analysis was performed directly on the running router using an interactive SSH session.

---

# Methodology

The following commands were used during the analysis:

```bash
ps

cat /proc/1/cmdline

cat /usr/etc/inittab

grep -n cfg_manager /usr/etc/init.d/rcS

ls -l /proc/<pid>/fd

netstat -an

cat /proc/<pid>/status
```

---

# Runtime Process List

The running services were identified using:

```bash
ps
```

Important processes observed during the analysis include:

| Process | Purpose |
|----------|---------|
| init | System initialization |
| cfg_manager | Central configuration daemon |
| monitorcfgmgr | Configuration monitor |
| boa | Web management server |
| dropbear | SSH server |
| dnsmasq | DNS forwarding |
| udhcpd | DHCP server |
| pppd | PPPoE client |
| wanduck | WAN monitoring |
| infosvr | ASUS device discovery |

---

# Central Configuration Service

The most important userspace process is:

```
cfg_manager
```

Unlike traditional Linux systems where configuration is usually stored in text files, this firmware centralizes configuration management through a dedicated daemon.

Responsibilities include:

- Configuration storage
- Configuration updates
- Service restart requests
- Runtime synchronization
- Firewall regeneration
- DNS configuration
- DHCP configuration

---

# Service Startup

During boot, `rcS` starts the configuration manager:

```text
/userfs/bin/cfg_manager &

/userfs/bin/monitorcfgmgr &
```

This indicates that all higher-level configuration depends on `cfg_manager`.

---

# IPC Communication

The analysis revealed that services communicate through a Unix Domain Socket.

Socket location:

```
/tmp/tcapi_sock
```

The socket was confirmed using:

```bash
netstat -an
```

Output:

```text
unix ... /tmp/tcapi_sock
```

The `cfg_manager` process keeps this socket open during normal operation.

---

# tcapi

Configuration changes are performed through the `tcapi` utility.

Available operations include:

```text
get
set
show
commit
save
read
readAll
```

The CLI communicates with `cfg_manager` through the Unix Domain Socket.

This architecture separates the user interface from the configuration engine.

---

# Web Management

The firmware uses:

```
Boa
```

instead of Apache or Nginx.

Boa provides:

- Web interface
- CGI execution
- Configuration pages
- tcWebApi integration

The web interface does not directly modify configuration files.

Instead, requests are forwarded to `cfg_manager` through the tcapi framework.

---

# SSH Service

Remote shell access is provided by:

```
Dropbear
```

Observations:

- Lightweight SSH implementation
- Password authentication
- Runtime host key loading
- Managed independently from the web interface

---

# Network Services

Networking is divided into multiple dedicated services.

## DHCP

```
udhcpd
```

Provides:

- Address allocation
- Lease management
- Client configuration

---

## DNS

```
dnsmasq
```

Provides:

- DNS forwarding
- Local caching
- DHCP integration

---

## PPP

```
pppd
```

Responsible for:

- PPPoE authentication
- WAN connectivity
- ISP communication

---

# Service Relationships

The runtime architecture can be summarized as follows.

```text
                     User
                      │
          ┌───────────┴───────────┐
          │                       │
          ▼                       ▼
      Web Browser              SSH Client
          │                       │
          ▼                       ▼
         Boa                  Dropbear
          │
          ▼
       tcWebApi
          │
          ▼
      libtcapi.so
          │
          ▼
    /tmp/tcapi_sock
          │
          ▼
      cfg_manager
          │
 ┌────────┼────────┬────────┐
 ▼        ▼        ▼        ▼
dnsmasq udhcpd  Firewall  pppd
```

---

# Key Findings

The service architecture demonstrates that the firmware is organized around a central management daemon.

Major observations:

- `cfg_manager` acts as the core service.
- Configuration changes are not written directly by applications.
- `tcapi` provides an abstraction layer.
- Services communicate using IPC rather than configuration files.
- Web and SSH interfaces are independent access methods.
- Networking functionality is distributed across specialized daemons.

---

# Commands Used

```bash
ps

netstat -an

cat /proc/1/cmdline

cat /proc/<pid>/status

ls -l /proc/<pid>/fd

grep cfg_manager /usr/etc/init.d/rcS
```

---

# Conclusion

The ASUS DSL-AC750 firmware follows a modular service architecture centered around `cfg_manager`.

Rather than allowing individual services to modify configuration independently, all configuration changes flow through a centralized management daemon using an IPC mechanism based on Unix Domain Sockets.

This architecture improves consistency while simplifying service synchronization and runtime configuration management.
