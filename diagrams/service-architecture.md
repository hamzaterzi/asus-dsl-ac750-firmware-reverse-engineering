# Service Architecture

```mermaid
flowchart TD

    Browser["Web Browser"] --> Boa["Boa Web Server"]
    SSH["SSH Client"] --> Dropbear["Dropbear SSH Server"]

    Boa --> tcWebApi["tcWebApi"]
    tcWebApi --> libtcapi["libtcapi.so"]

    libtcapi --> Socket["/tmp/tcapi_sock"]

    Socket --> cfg_manager["cfg_manager"]

    cfg_manager --> dnsmasq["dnsmasq"]
    cfg_manager --> udhcpd["udhcpd"]
    cfg_manager --> pppd["pppd"]
    cfg_manager --> Firewall["iptables / Firewall"]
```

## Description

This diagram illustrates the runtime service architecture discovered during the reverse engineering process.

The web interface communicates with `cfg_manager` through `tcWebApi` and `libtcapi.so`. Communication is performed over the Unix Domain Socket located at `/tmp/tcapi_sock`.

`cfg_manager` acts as the central management daemon and coordinates runtime services such as DNS, DHCP, PPPoE, and firewall configuration.
