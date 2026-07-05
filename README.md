# ASUS DSL-AC750 Firmware Reverse Engineering

> A hands-on reverse engineering project focused on understanding the internal architecture of an Embedded Linux router through live runtime analysis, firmware exploration, and system-level research.

![Platform](https://img.shields.io/badge/Platform-Embedded%20Linux-2ea44f)
![Firmware](https://img.shields.io/badge/Firmware-ASUS%20DSL--AC750-blue)
![Shell](https://img.shields.io/badge/Shell-BusyBox-orange)
![Research](https://img.shields.io/badge/Type-Reverse%20Engineering-red)
![Status](https://img.shields.io/badge/Status-Active-success)

---

# Research Focus

This repository documents the reverse engineering of an ASUS DSL-AC750 router running an Embedded Linux firmware.

Instead of only extracting the firmware image, this research is performed directly on a live device through an SSH shell, allowing runtime observation of processes, services, configuration changes, filesystem layout, and inter-process communication.

The primary goal is to understand how a consumer networking device operates internally by analyzing its boot process, runtime architecture, configuration database, networking stack, and management services.

---

# Table of Contents

- Overview
- Research Objectives
- Lab Environment
- Technologies
- System Architecture
- Research Progress
- Repository Structure
- Key Findings
- Future Work
- Disclaimer
- License

---

# Overview

During this research the following components are being analyzed:

- Embedded Linux operating system
- BusyBox userspace
- Boot sequence
- Flash memory layout
- Runtime filesystem
- Configuration database
- tcapi framework
- cfg_manager daemon
- Unix Domain Socket IPC
- Dropbear SSH server
- Boa Web Server
- DHCP/DNS services
- Firewall architecture
- Persistent configuration storage

The project aims to document every stage of the analysis in a structured and reproducible way.

---

# Research Objectives

- Understand the Embedded Linux boot process
- Analyze the firmware filesystem
- Explore MTD flash partitions
- Document SquashFS and JFFS2 usage
- Reverse engineer the configuration database
- Analyze tcapi communication
- Understand cfg_manager responsibilities
- Discover IPC mechanisms
- Analyze runtime services
- Study firewall implementation
- Document service dependencies
- Produce high-quality technical documentation

---

# Lab Environment

## Hardware

- ASUS DSL-AC750
- ASUS DSL-AC51 firmware family

## Operating System

- Embedded Linux
- BusyBox

## Access

- SSH (Dropbear)

---

# Technologies

- Embedded Linux
- BusyBox
- Dropbear
- Boa Web Server
- tcapi
- cfg_manager
- Unix Domain Sockets
- MTD
- SquashFS
- JFFS2
- PPPoE
- dnsmasq
- udhcpd
- iptables
- OpenSSL

---

# System Architecture

```text
                     Administrator
                           │
                           ▼
                     SSH (Dropbear)
                           │
                           ▼
                  Embedded Linux Shell
                           │
                           ▼
                      tcapi CLI
                           │
                           ▼
                     libtcapi.so
                           │
            Unix Domain Socket (IPC)
                  /tmp/tcapi_sock
                           │
                           ▼
                     cfg_manager
        ┌────────────┬─────────────┬────────────┐
        │            │             │            │
        ▼            ▼             ▼            ▼
      Boa         dnsmasq       udhcpd      Firewall
        │                            │           │
        ▼                            ▼           ▼
    Web GUI                    DHCP / DNS    iptables
```

---

# Research Progress

## Completed

- SSH access to the router
- Runtime process analysis
- Embedded Linux filesystem exploration
- Flash partition mapping
- MTD analysis
- SquashFS identification
- JFFS2 writable partition analysis
- Boot initialization research
- Service discovery
- tcapi analysis
- cfg_manager analysis
- IPC discovery
- Unix Domain Socket analysis
- Runtime configuration analysis
- Dropbear analysis

## In Progress

- Firewall architecture
- Service dependency mapping
- Configuration persistence
- Runtime service orchestration
- Boot sequence documentation
- Configuration workflow analysis

---

# Repository Structure

```
asus-dsl-ac750-firmware-reverse-engineering
│
├── README.md
│
├── docs
│   ├── 01-filesystem-and-flash-layout.md
│   ├── 02-boot-process.md
│   ├── 03-service-architecture.md
│   ├── 04-tcapi-analysis.md
│   ├── 05-ipc-analysis.md
│   ├── 06-configuration-database.md
│   ├── 07-dropbear-analysis.md
│   ├── 08-firewall-analysis.md
│   ├── 09-runtime-analysis.md
│   └── 10-final-architecture.md
│
├── images
│
├── diagrams
│
└── scripts
```

---

# Key Findings

Current findings indicate that the firmware architecture is centered around **cfg_manager**, which acts as the primary configuration daemon.

Configuration changes are performed through **tcapi**, which communicates with cfg_manager using a Unix Domain Socket located at:

```
/tmp/tcapi_sock
```

The firmware uses:

- BusyBox userland
- SquashFS read-only root filesystem
- JFFS2 writable persistent storage
- MTD flash partitions
- Dropbear SSH server
- Boa Web Server
- dnsmasq
- udhcpd
- iptables firewall

This architecture allows runtime configuration updates without directly modifying the read-only firmware image.

---

# Future Work

The following topics will be explored in future research:

- Bootloader analysis
- Firmware extraction
- Flash image reconstruction
- Kernel module analysis
- Firewall rule generation
- NAT implementation
- tcapi protocol internals
- IPC message format
- Service dependency graph
- Network packet analysis
- Web interface reverse engineering

---

# Disclaimer

This project is performed exclusively on personally owned hardware within a private laboratory environment.

No unauthorized systems, networks, or third-party devices are involved.

Sensitive information such as passwords, PPPoE credentials, SSH host keys, MAC addresses, serial numbers, and other private configuration values are intentionally excluded from this repository.

The purpose of this repository is educational and research-oriented.

---

# License

This repository is published for educational and research purposes.

---

## Repository Status

🚧 Active Research

This repository is continuously updated as new reverse engineering findings are discovered and documented.

## Project Status

✅ Firmware extraction completed

✅ Boot process documented

✅ Service architecture documented

✅ tcapi analysis completed

✅ IPC analysis completed

✅ Configuration database documented

✅ Firewall architecture documented

✅ cfg_manager reverse engineering completed

This repository documents the complete reverse engineering process of the ASUS DSL-AC750 firmware configuration subsystem.

Current status: **Completed (v1.0)**
