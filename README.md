# newton
## 环境准备
Ubuntu Server 16.04 LTS  
| 节点  | 地址规划 |
| ---------- | -----------|
| Controller Node   | 192.168.1.11   |
| Compute Node   | 192.168.1.21   |
| Block Storage Node   | 192.168.1.31   |
| Object Storage Node 1   | 192.168.1.41   |
| Object Storage Node 2   | 192.168.1.42   |

网络配置  
vi /etc/network/interfaces
```
auto ens3
iface ens3 inet static
        address 192.168.1.11
        netmask 255.255.255.0
        network 192.168.1.0
        broadcast 192.168.1.255
        gateway 192.168.1.1
        dns-nameservers 192.168.1.1
```

网络名称解析  
vi /etc/hosts
```
# controller
192.168.1.11       controller

# compute
192.168.1.21       compute

# block
192.168.1.31       block

# object1
192.168.1.41       object1

# object2
192.168.1.42       object2
```
