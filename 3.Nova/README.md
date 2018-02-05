Nova部署
---

创建nova数据库

> mysql -u root -p

```
CREATE DATABASE nova;
CREATE DATABASE nova_api;

GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'asd';

exit;
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
> nova-manage db sync  
> nova-manage api_db sync  

重启nova服务
> service nova-api restart  
> service nova-consoleauth restart  
> service nova-scheduler restart  
> service nova-conductor restart  
> service nova-novncproxy restart  
