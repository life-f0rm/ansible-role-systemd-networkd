# ansible-role-systemd-networkd

Ansible role to manage systemd networkd

This Ansible role configures systemd-networkd to manage network interfaces, replacing Netplan.

## Features

- Configure physical interfaces with MAC address matching and renaming (`.link` files)
- Create VLAN interfaces (`.netdev` files)
- Create bridge interfaces (`.netdev` files) with VLAN-aware bridging support
- Create bond interfaces (`.netdev` files) with LACP support
- Configure network settings including IP addresses, DHCP, routes (`.network` files)
- Support for VLAN-aware bridges with access ports and trunk ports
- Configure loopback interface
- Automatically remove Netplan configuration files

## Role Variables

### Main Configuration

- `networkd_config`: Dictionary containing all network configuration
  - `links`: Physical interface MAC address matching and renaming
  - `vlans`: VLAN interface definitions (for L3 VLAN subinterfaces)
  - `bonds`: Bond interface definitions (for link aggregation)
  - `bridges`: Bridge interface definitions (supports VLAN-aware bridging)
  - `networks`: Network configuration for all interfaces (supports BridgeVLAN for access/trunk ports)
  - `loopback`: Loopback interface configuration

- `networkd_enable_service`: Enable and start systemd-networkd service (default: `true`)
- `networkd_enable_resolved`: Enable and start systemd-resolved service (default: `true`)
- `networkd_remove_netplan`: Remove Netplan configuration files (default: `false`)

## Configuration Structure

### Links (Physical Interfaces)

```yaml
networkd_config:
  links:
    internal0:
      MACAddress: "a8:b8:e0:0a:21:37"
      Name: "internal0"
```

### VLANs (L3 VLAN Subinterfaces)

```yaml
networkd_config:
  vlans:
    t3_users:
      id: 103
    default:
      id: 182
```

### Bonds (Link Aggregation)

```yaml
networkd_config:
  bonds:
    bond0:
      Mode: 802.3ad
      MIIMonitorSec: 1s
      LACPTransmitRate: fast
```

### Bridges (VLAN-Aware)

```yaml
networkd_config:
  bridges:
    br0:
      vlan_filtering: true
      default_pvid: 99
      stp: true
      priority: 0
```

### Networks

#### Access Port (Single VLAN, untagged)

```yaml
networkd_config:
  networks:
    internal0:
      Match:
        Name: "internal0"
      Bridge: "br0"
      BridgeVLAN:
        - vlan: 103
          pvid: true
          egress_untagged: true
```

#### Trunk Port (Multiple VLANs, tagged)

```yaml
networkd_config:
  networks:
    bond0:
      Match:
        Name: "bond0"
      Bridge: "br0"
      BridgeVLAN:
        - vlan: 99
          pvid: true
          egress_untagged: true
        - vlan: 103
        - vlan: 182
```

#### Bond Slave

```yaml
networkd_config:
  networks:
    internal2:
      Match:
        Name: "internal2"
      Bond: "bond0"
```

#### VLAN L3 Interface

```yaml
networkd_config:
  networks:
    t3_users:
      Match:
        Name: "t3_users"
      Address:
        - "192.168.1.254/24"
      ConfigureWithoutCarrier: "yes"
```

#### Bridge with VLAN Interfaces

```yaml
networkd_config:
  networks:
    br0:
      Match:
        Name: "br0"
      LinkLocalAddressing: "no"
      ConfigureWithoutCarrier: "yes"
      VLAN:
        - "t3_users"
        - "default"
      BridgeVLAN:
        - vlan: 99
        - vlan: 103
        - vlan: 182
```

#### DHCP Interface

```yaml
networkd_config:
  networks:
    isp0:
      Match:
        Name: "isp0"
      DHCP: "yes"
      DHCPv6: "yes"
```

### Loopback

```yaml
networkd_config:
  loopback:
    Match:
      Name: "lo"
    Address:
      - "127.0.0.1/8"
      - "::1/128"
      - "{{ loopback_address }}/32"
```

## Example Playbook

```yaml
- name: Configure systemd-networkd
  hosts: routers
  become: true
  roles:
    - networkd
```

## Dependencies

None

## License

MIT
