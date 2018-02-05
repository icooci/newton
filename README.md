# newton
## 环境准备

OS: Ubuntu Server 16.04 LTS  

| 节点 | 主机名 | 地址规划 |
| ---- | ----- | ------- |
| Controller Node | controller | 192.168.1.11 |
| Compute Node | compute | 192.168.1.21 |
| Block Storage Node | block | 192.168.1.31 |
| Object Storage Node 1 | object1 | 192.168.1.41 |
| Object Storage Node 2 | object2 | 192.168.1.42 |

网络配置  
> vi /etc/network/interfaces  
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
控制节点部署NTP服务  
> apt install chrony  
> vi /etc/chrony/chrony.conf  
```diff
+allow 192.168.1.0/24
```
> service chrony restart  

其他节点部署NTP服务
> apt install chrony
> vi /etc/chrony/chrony.conf
```diff
- pool 2.debian.pool.ntp.org offline iburst
+ server controller iburst
```
> service chrony restart  

验证NTP同步信息  
> chronyc sources

