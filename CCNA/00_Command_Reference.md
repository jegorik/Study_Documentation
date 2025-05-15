# CCNA Command Reference

A comprehensive reference for essential Cisco IOS commands organized by topic.

## Basic Device Management
+--------------------------------------+---------------------------------+-------------------+
| Command                              | Description                     | Mode              |
|--------------------------------------+---------------------------------+-------------------|
| `enable`                             | Enter privileged EXEC mode      | User EXEC         |
| `configure terminal`                 | Enter global configuration mode | Privileged EXEC   |
| `hostname [name]`                    | Set device hostname             | Global config     |
| `show running-config`                | Display current configuration   | Privileged EXEC   |
| `show startup-config`                | Display saved configuration     | Privileged EXEC   |
| `copy running-config startup-config` | Save configuration              | Privileged EXEC   |
| `reload`                             | Reboot device                   | Privileged EXEC   |
| `no shutdown`                        | Enable interface                | Interface config  |
| `shutdown`                           | Disable interface               | Interface config  |
+--------------------------------------+---------------------------------+-------------------+
## Interface Configuration
+-----------------------------------+-----------------------------------+-------------------+
| Command                           | Description                       | Mode              |
|-----------------------------------+-----------------------------------+-------------------|
| `interface [type][slot/number]`   | Enter interface configuration mode| Global config     |
| `description [text]`              | Set interface description         | Interface config  |
| `ip address [ip-address] [mask]`  | Set IPv4 address                  | Interface config  |
| `ipv6 address [ipv6-addr/prefix]` | Set IPv6 address                  | Interface config  |
| `speed [10/100/1000/auto]`        | Set interface speed               | Interface config  |
| `duplex [auto/full/half]`         | Set duplex mode                   | Interface config  |
| `show interfaces`                 | Display interface status          | Privileged EXEC   |
| `show ip interface brief`         | Display IP interface summary      | Privileged EXEC   |
+-----------------------------------+-----------------------------------+-------------------+

## Switching
+--------------------------------------------+----------------------------------------+------------------+
| Command                                    | Description                            | Mode             |
|--------------------------------------------+----------------------------------------+------------------|
| `vlan [vlan-id]`                           | Create VLAN and enter VLAN config mode | Global config    |
| `name [vlan-name]`                         | Set VLAN name                          | VLAN config      |
| `switchport mode access`                   | Set port to access mode                | Interface config |
| `switchport access vlan [vlan-id]`         | Assign port to VLAN                    | Interface config |
| `switchport mode trunk`                    | Set port to trunk mode                 | Interface config |
| `switchport trunk allowed vlan [vlan-list]`| Set allowed VLANs on trunk             | Interface config |
| `switchport trunk native vlan [vlan-id]`   | Set native VLAN for trunk              | Interface config |
| `show vlan`                                | Display VLAN information               | Privileged EXEC  |
| `show interfaces trunk`                    | Display trunk ports information        | Privileged EXEC  |
+--------------------------------------------+----------------------------------------+------------------+

## Spanning Tree Protocol
+----------------------------------------------+--------------------------------+------------------+
| Command                                      | Description                    | Mode             |
|----------------------------------------------+--------------------------------+------------------|
| `spanning-tree mode [pvst/rapid-pvst/mst]`   | Set STP mode                   | Global config    |
| `spanning-tree vlan [vlan-id] root primary`  | Make switch root bridge        | Global config    |
| `spanning-tree vlan [vlan-id] root secondary`| Make switch backup root bridge | Global config    |
| `spanning-tree portfast`                     | Enable PortFast on access port | Interface config |
| `spanning-tree bpduguard enable`             | Enable BPDU guard on port      | Interface config |
| `show spanning-tree`                         | Display STP information        | Privileged EXEC  |
+----------------------------------------------+--------------------------------+------------------+

## Routing
+----------------------------------------------+----------------------------------+----------------+
| Command                                      | Description                      | Mode           |
|----------------------------------------------+----------------------------------+----------------|
| `ip routing`                                 | Enable IP routing                | Global config  |
| `ip route [network] [mask] [next-hop/iface]` | Configure static route           | Global config  |
| `router ospf [process-id]`                   | Enter OSPF configuration mode    | Global config  |
| `network [net] [wildcard-mask] area [id]`    | Enable OSPF on interfaces        | OSPF config    |
| `show ip route`                              | Display routing table            | Privileged EXEC|
| `show ip ospf neighbor`                      | Display OSPF neighbors           | Privileged EXEC|
+----------------------------------------------+----------------------------------+----------------+

## ACLs
+------------------------------------------------------------------------+----------------------------------+------------------+
| Command                                                                | Description                      | Mode             |
|------------------------------------------------------------------------+----------------------------------+------------------|
| `access-list [number] [permit/deny] [source] [wildcard]`               | Create standard ACL              | Global config    |
| `access-list [num] [permit/deny] [protocol] [src] [wild] [dst] [wild]` | Create extended ACL              | Global config    |
| `ip access-group [number] [in/out]`                                    | Apply ACL to interface           | Interface config |
| `show access-lists`                                                    | Display ACLs                     | Privileged EXEC  |
| `show ip interface`                                                    | Show ACLs applied to interface   | Privileged EXEC  |
+------------------------------------------------------------------------+----------------------------------+------------------+

## NAT
+----------------------------------------------------------------------------+--------------------------+------------------+
| Command                                                                    | Description              | Mode             |
|----------------------------------------------------------------------------+--------------------------+------------------|
| `ip nat inside`                                                            | Mark interface as inside | Interface config |
| `ip nat outside`                                                           | Mark interface as outside| Interface config |
| `ip nat inside source static [local-ip] [global-ip]`                       | Configure static NAT     | Global config    |
| `ip nat inside source list [acl-number] pool [name]`                       | Configure dynamic NAT    | Global config    |
| `ip nat inside source list [acl-number] interface [type][number] overload` | Configure PAT            | Global config    |
| `show ip nat translations`                                                 | Display NAT translations | Privileged EXEC  |
+----------------------------------------------------------------------------+--------------------------+------------------+

## DHCP
+------------------------------------------------+-----------------------------+------------------+
| Command                                        | Description                 | Mode             |
|------------------------------------------------+-----------------------------+------------------|
| `ip dhcp pool [name]`                          | Create DHCP pool            | Global config    |
| `network [network] [mask]`                     | Define network for DHCP     | DHCP config      |
| `default-router [ip-address]`                  | Define default gateway      | DHCP config      |
| `dns-server [ip-address]`                      | Define DNS server           | DHCP config      |
| `ip dhcp excluded-address [start-ip] [end-ip]` | Exclude addresses from DHCP | Global config    |
| `ip helper-address [dhcp-server-ip]`           | Configure DHCP relay        | Interface config |
| `show ip dhcp binding`                         | Display DHCP bindings       | Privileged EXEC  |
+------------------------------------------------+-----------------------------+------------------+

## Security
+--------------------------------------+------------------------------+----------------+
| Command                              | Description                  | Mode           |
|--------------------------------------+------------------------------+----------------|
| `username [name] password [password]`| Create local user            | Global config  |
| `enable password [password]`         | Set enable password          | Global config  |
| `enable secret [password]`           | Set encrypted enable password| Global config  |
| `service password-encryption`        | Encrypt passwords in config  | Global config  |
| `line console 0`                     | Enter console line config    | Global config  |
| `line vty 0 15`                      | Enter VTY line config        | Global config  |
| `login local`                        | Use local authentication     | Line config    |
| `transport input ssh`                | Allow SSH connections only   | Line config    |
| `crypto key generate rsa`            | Generate RSA key for SSH     | Global config  |
+--------------------------------------+------------------------------+----------------+

## Device Monitoring
+---------------------------+-----------------------------+---------------------+
| Command                   | Description                 | Mode                |
|---------------------------+-----------------------------+---------------------|
| `show version`            | Display device information  | Privileged EXEC     |
| `show cdp neighbors`      | Display CDP neighbors       | Privileged EXEC     |
| `show lldp neighbors`     | Display LLDP neighbors      | Privileged EXEC     |
| `show ip protocols`       | Display routing protocols   | Privileged EXEC     |
| `show processes cpu`      | Display CPU utilization     | Privileged EXEC     |
| `show memory`             | Display memory utilization  | Privileged EXEC     |
| `ping [destination]`      | Send ICMP echo request      | User/Privileged EXEC|
| `traceroute [destination]`| Trace route to destination  | User/Privileged EXEC|
| `debug [function]`        | Enable debugging            | Privileged EXEC     |
| `no debug [function]`     | Disable debugging           | Privileged EXEC     |
+---------------------------+-----------------------------+---------------------+
