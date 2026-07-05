# Service Architecture

```mermaid
flowchart TD

    Admin["Administrator"]

    Admin --> Browser["Web Browser"]
    Admin --> SSH["SSH Client"]

    Browser --> Boa["Boa Web Server"]
    SSH --> Dropbear["Dropbear SSH Server"]

    Boa --> tcWebApi["tcWebApi"]
    Dropbear --> Shell["Embedded Linux Shell"]

    Shell --> CLI["tcapi CLI"]

    tcWebApi --> libtcapi["libtcapi.so"]
    CLI --> libtcapi

    libtcapi --> Socket["Unix Domain Socket<br>/tmp/tcapi_sock"]

    Socket --> CFG["cfg_manager"]

    CFG --> DHCP["udhcpd<br>DHCP Server"]
    CFG --> DNS["dnsmasq<br>DNS Forwarder"]
    CFG --> PPP["pppd<br>PPPoE Client"]
    CFG --> FW["iptables<br>Firewall / NAT"]
    CFG --> Web["Boa Runtime Control"]
    CFG --> DB["Configuration Database"]

    DB --> Romfile["romfile.cfg"]
    Romfile --> Flash["MTD romfile partition"]
```

## Description

This diagram illustrates the runtime service architecture discovered during the reverse engineering process.

The router exposes two main administrative paths:

- Web-based management through Boa and `tcWebApi`
- Shell-based management through Dropbear SSH and `tcapi`

Both paths eventually use `libtcapi.so` and communicate with `cfg_manager` through the Unix Domain Socket located at `/tmp/tcapi_sock`.

`cfg_manager` acts as the central service orchestration and configuration daemon. It coordinates runtime services such as DHCP, DNS, PPPoE, firewall/NAT, web management, and persistent configuration storage.
