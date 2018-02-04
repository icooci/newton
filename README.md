# newton
openstack newton
环境准备
Ubuntu Server 16.04 LTS x 3
地址规划
Controller Node   192.168.1.11
Compute Node    192.168.1.21
Block Storage Node    192.168.1.31
Object Storage Node 1   192.168.1.41
Object Storage Node 1   192.168.1.42
cat /var/lib/neutron/dhcp/8fda183f-4e8d-406e-88b1-bd629c7eb3b1/addn_hosts
cat /var/lib/neutron/dhcp/8fda183f-4e8d-406e-88b1-bd629c7eb3b1/opts
啊实打实的
啊实打实的111
ss是Socket Statistics的缩写。

顾名思义，ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。
编辑网络配置
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
