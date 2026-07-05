# Filesystem and Flash Layout Analysis

## Overview

The first step in understanding an embedded Linux device is identifying how its firmware is stored and how the operating system is organized at runtime.

This document analyzes the flash memory layout, mounted filesystems, and runtime storage architecture of the ASUS DSL-AC750.

The analysis was performed directly on the running device using an interactive SSH session.

---

# Flash Memory Layout

The router stores its firmware inside MTD (Memory Technology Device) partitions.

The partition table was obtained using:

```bash
cat /proc/mtd
```

Output:

```text
dev:    size   erasesize  name

mtd0  bootloader
mtd1  romfile
mtd2  kernel
mtd3  rootfs
mtd4  tclinux
mtd5  jffs2
mtd6  reservearea
```

## Partition Description

| Partition | Purpose |
|-----------|----------|
| bootloader | Device bootloader |
| romfile | Persistent configuration |
| kernel | Linux kernel |
| rootfs | Root filesystem |
| tclinux | Combined firmware image |
| jffs2 | Writable persistent storage |
| reservearea | Reserved vendor storage |

---

# Runtime Filesystem

The mounted filesystems were inspected using:

```bash
mount
```

Important results:

```text
/dev/mtdblock3 on / type squashfs (ro)

/dev/mtdblock5 on /jffs type jffs2 (rw)
```

This immediately reveals an architecture commonly used by embedded Linux systems.

## Root Filesystem

The root filesystem is mounted as:

```
SquashFS
```

Characteristics:

- Read-only
- Compressed
- Stored inside flash memory
- Cannot be modified during normal operation

---

## Writable Storage

A separate writable partition is mounted as:

```
JFFS2
```

This partition stores runtime data and persistent configuration generated after the device boots.

---

# Storage Usage

Filesystem usage was inspected using:

```bash
df -h
```

Example:

```text
Filesystem          Size    Used   Available

rootfs             10 MB    100%

jffs2               1 MB      21%
```

Observations:

- The firmware image occupies the entire SquashFS partition.
- Runtime modifications are stored separately inside JFFS2.
- The root filesystem remains immutable during normal operation.

---

# Directory Layout

The root directory contains the standard embedded Linux hierarchy.

Example:

```
/
├── bin
├── boaroot
├── dev
├── jffs
├── lib
├── proc
├── sbin
├── sys
├── tmp
├── userfs
├── usr
└── var
```

Several directories such as `/etc`, `/home`, `/var`, and `/mnt` are symbolic links pointing into the temporary filesystem.

This indicates that part of the runtime environment is dynamically generated during boot.

---

# Runtime Architecture

The storage architecture can be summarized as follows:

```text
Flash Memory
│
├── Bootloader
├── Kernel
├── SquashFS (Read Only)
│       │
│       ├── /bin
│       ├── /sbin
│       ├── /usr
│       └── /lib
│
└── JFFS2 (Writable)
        │
        └── Persistent Configuration
```

---

# Key Findings

The filesystem architecture follows a common embedded Linux design:

- SquashFS provides a compressed read-only firmware image.
- JFFS2 provides writable persistent storage.
- Runtime files are generated dynamically during boot.
- Symbolic links are extensively used to relocate writable directories into temporary memory.
- The firmware separates immutable operating system components from runtime configuration.

This design minimizes flash writes while allowing configuration changes without modifying the firmware image.

---

# Commands Used

```bash
cat /proc/mtd

cat /proc/partitions

mount

df -h

ls -la /

find / -name inittab

find / -name rcS
```

---

# Conclusion

The ASUS DSL-AC750 firmware follows a traditional embedded Linux architecture based on read-only firmware images and separate writable flash storage.

Understanding this storage layout provides the foundation for analyzing the boot process, configuration management, service initialization, and runtime behavior explored in the following sections of this research.
