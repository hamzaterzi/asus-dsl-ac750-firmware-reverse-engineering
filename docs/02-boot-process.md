# Boot Process Analysis

## Overview

After understanding the filesystem layout, the next step was to identify how the ASUS DSL-AC750 boots and initializes its runtime environment.

The objective of this analysis was to determine which process starts first, how the initialization scripts are executed, and how the system services are launched.

The analysis was performed directly on the running router through an SSH session.

---

# Analysis Methodology

The following commands were executed during the investigation:

```bash
cat /proc/1/cmdline

cat /usr/etc/inittab

cat /tmp/etc/inittab

head -120 /usr/etc/init.d/rcS

ps

mount
```

These commands allowed the complete boot chain to be reconstructed.

---

# Process 1

The first running process was identified using:

```bash
cat /proc/1/cmdline
```

Result:

```text
init
```

This confirms that the Linux kernel transfers control to the standard userspace initialization process after booting.

---

# init Configuration

The initialization configuration is stored in:

```text
/usr/etc/inittab
```

The file contains:

```text
::sysinit:/usr/etc/init.d/rcS

::askfirst:/sbin/getty -L ttyS0 115200 vt100
```

This indicates that the router executes the `rcS` initialization script during system startup.

---

# rcS Initialization Script

The boot script was inspected using:

```bash
head -120 /usr/etc/init.d/rcS
```

The script performs several initialization tasks, including:

- Setting the initial system date
- Loading configuration variables
- Creating runtime directories
- Mounting filesystems
- Creating device nodes
- Loading kernel modules
- Preparing the runtime environment

The script also prepares temporary directories such as:

```text
/tmp
/var
/dev
/home
```

before starting higher-level services.

---

# Service Initialization

Further inspection of the initialization script revealed that the central configuration daemon is started during boot.

Relevant section:

```text
/userfs/bin/cfg_manager &

/userfs/bin/monitorcfgmgr &
```

This indicates that `cfg_manager` is one of the first userspace services launched after the basic operating system initialization.

---

# Runtime Service Discovery

Running processes were inspected using:

```bash
ps
```

Several important services were identified:

| Service | Purpose |
|----------|---------|
| cfg_manager | Central configuration daemon |
| monitorcfgmgr | Configuration monitoring |
| boa | Web management interface |
| dropbear | SSH server |
| dnsmasq | DNS forwarding |
| udhcpd | DHCP server |
| pppd | PPPoE client |

This confirms that the router follows a service-oriented architecture where `cfg_manager` coordinates configuration while other daemons provide network functionality.

---

# Boot Sequence

The reconstructed boot process is illustrated below.

```text
Power On
    │
    ▼
Bootloader
    │
    ▼
Linux Kernel
    │
    ▼
init (PID 1)
    │
    ▼
/usr/etc/inittab
    │
    ▼
rcS
    │
    ├── Mount filesystems
    ├── Create device nodes
    ├── Load kernel modules
    ├── Prepare runtime directories
    │
    ▼
cfg_manager
    │
    ├── monitorcfgmgr
    ├── boa
    ├── dropbear
    ├── dnsmasq
    ├── udhcpd
    └── pppd
```

---

# Key Findings

The boot process follows the classic Embedded Linux initialization model.

Key observations include:

- `init` is the first userspace process.
- `inittab` starts the `rcS` initialization script.
- `rcS` prepares the runtime environment before launching services.
- `cfg_manager` acts as the primary management daemon.
- Network services are started only after the system initialization has completed.

---

# Commands Used

```bash
cat /proc/1/cmdline

cat /usr/etc/inittab

cat /tmp/etc/inittab

head -120 /usr/etc/init.d/rcS

ps

mount
```

---

# Conclusion

The boot process demonstrates that the ASUS DSL-AC750 firmware uses a traditional Embedded Linux initialization sequence.

Rather than launching services independently, the firmware first prepares the operating environment through `rcS` and then starts the configuration management subsystem, which subsequently controls the runtime behavior of the device.
