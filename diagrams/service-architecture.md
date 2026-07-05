# Service Architecture

```mermaid
flowchart TD

    Admin["Administrator"]

    Admin --> Browser["Web Browser"]
    Admin --> SSH["SSH Client"]

    Browser --> Boa["Boa Web Server"]
    SSH --> Dropbear["Dropbear SSH Server"]

    Boa --> tcWebApi["tcWebApi"]

    Dropbear --> Shell["Linux Shell"]
    Shell --> CLI["tcapi CLI"]

    tcWebApi --> Lib["libtcapi.so"]
    CLI --> Lib

    Lib --> Socket["/tmp/tcapi_sock"]

    Socket --> CFG["cfg_manager"]

    CFG --> DHCP["udhcpd"]
    CFG --> DNS["dnsmasq"]
    CFG --> PPP["pppd"]
    CFG --> FW["iptables"]

    CFG --> Config["Configuration Database"]

    Config --> Rom["romfile.cfg"]

    Rom --> Flash["MTD: romfile"]
```

## Description

This diagram summarizes the runtime service architecture discovered during the reverse engineering of the ASUS DSL-AC750 firmware.

Both the web interface and the command-line interface ultimately communicate with the `cfg_manager` daemon through `libtcapi.so` and the Unix Domain Socket located at `/tmp/tcapi_sock`.

`cfg_manager` is responsible for coordinating runtime services and managing persistent configuration stored in the `romfile` flash partition.
