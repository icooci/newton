Nova部署 - 控制节点
---

创建nova数据库

> mysql -u root -p

```
CREATE DATABASE nova_api;
CREATE DATABASE nova;

GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'asd';

^D
```

加载admin变量
>. admin-openrc

创建nova用户
> openstack user create --domain default --password-prompt nova

为nova用户分配admin角色
> openstack role add --project service --user nova admin  
`本条命令无回显`

创建nova服务实体
> openstack service create --name nova --description "OpenStack Compute" compute

创建nova服务API Endpoint
> openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1/%\(tenant_id\)s  
> openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1/%\(tenant_id\)s  
> openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1/%\(tenant_id\)s

安装nova软件包
> apt install nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler

编辑nova配置
> vi /etc/nova/nova.conf  

```bash
[DEFAULT]
dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
# log-dir=/var/log/nova
state_path=/var/lib/nova
force_dhcp_release=True
verbose=True
ec2_private_dns_show_ip=True
enabled_apis=osapi_compute,metadata
transport_url = rabbit://openstack:asd@controller
auth_strategy = keystone
my_ip = 192.168.1.11
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver


[database]
# connection=sqlite:////var/lib/nova/nova.sqlite
connection = mysql+pymysql://nova:asd@controller/nova

[api_database]
# connection=sqlite:////var/lib/nova/nova.sqlite
connection = mysql+pymysql://nova:asd@controller/nova_api

[oslo_concurrency]
# lock_path=/var/lock/nova
lock_path = /var/lib/nova/tmp

[libvirt]
use_virtio_for_bridges=True

[wsgi]
api_paste_config=/etc/nova/api-paste.ini

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = asd

[vnc]
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip

[glance]
api_servers = http://controller:9292

```

初始化nova数据库
> nova-manage api_db sync  
> nova-manage db sync  

重启nova服务
> service nova-api restart  
> service nova-consoleauth restart  
> service nova-scheduler restart  
> service nova-conductor restart  
> service nova-novncproxy restart  

验证操作
---

加载admin变量
> . admin-openrc

查询计算服务
> openstack compute service list

服务器返回结果如下:
```
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  5 | nova-consoleauth | controller | internal | enabled | up    | 2018-02-05T12:55:06.000000 |
|  6 | nova-scheduler   | controller | internal | enabled | up    | 2018-02-05T12:55:06.000000 |
|  7 | nova-conductor   | controller | internal | enabled | up    | 2018-02-05T12:55:06.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
```


Nova部署 - 计算节点
---

安装nova-compute软件包
> apt install nova-compute

编辑nova配置文件
> vi /etc/nova/nova.conf

```bash
[DEFAULT]
dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
# log-dir=/var/log/nova
state_path=/var/lib/nova
force_dhcp_release=True
verbose=True
ec2_private_dns_show_ip=True
enabled_apis=osapi_compute,metadata
transport_url = rabbit://openstack:asd@controller
auth_strategy = keystone
my_ip = 192.168.1.21
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[database]
connection=sqlite:////var/lib/nova/nova.sqlite

[api_database]
connection=sqlite:////var/lib/nova/nova.sqlite

[oslo_concurrency]
# lock_path=/var/lock/nova
lock_path = /var/lib/nova/tmp

[libvirt]
use_virtio_for_bridges=True

[wsgi]
api_paste_config=/etc/nova/api-paste.ini

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = asd

[vnc]
enabled = True
vncserver_listen = 0.0.0.0
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html

[glance]
api_servers = http://controller:9292
```

检测硬件虚拟化支持

> egrep -c '(vmx|svm)' /proc/cpuinfo

如果返回结果大于1，则表明此计算节点支持硬件加速虚拟化，不需要额外配置
否则需要将libvirt使用的hypervisor由kvm改为qemu

> vi /etc/nova/nova-compute.conf
```diff
[DEFAULT]
compute_driver=libvirt.LibvirtDriver
[libvirt]
- virt_type=kvm
+ virt_type=qemu
```

验证操作(在控制节点上操作)
---

加载admin变量
> . admin-openrc

查询计算服务
> openstack compute service list

服务器返回结果如下:
```
+----+------------------+------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  5 | nova-consoleauth | controller | internal | enabled | up    | 2018-02-05T13:19:27.000000 |
|  6 | nova-scheduler   | controller | internal | enabled | up    | 2018-02-05T13:19:26.000000 |
|  7 | nova-conductor   | controller | internal | enabled | up    | 2018-02-05T13:19:27.000000 |
|  8 | nova-compute     | compute    | nova     | enabled | up    | 2018-02-05T13:19:27.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
```
