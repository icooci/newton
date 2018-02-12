## Neutron OVS部署 - 控制节点

创建neutron数据库
> mysql -u root -p

```
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'asd';
EXIT;
```

加载admin变量
> . admin-openrc

创建neutron用户
> openstack user create --domain default --password-prompt neutron

为neutron用户分配admin角色
> openstack role add --project service --user neutron admin  
> `本条命令无回显`

创建neutron服务实体
> openstack service create --name neutron --description "OpenStack Networking" network

创建neutron服务API Endpoint
> openstack endpoint create --region RegionOne network public http://controller:9696  
> openstack endpoint create --region RegionOne network internal http://controller:9696  
> openstack endpoint create --region RegionOne network admin http://controller:9696  

<br />

## 网络类型: Self-service 网络

安装neutron及OVS软件包
> apt install neutron-server neutron-plugin-ml2 neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent neutron-openvswitch-agent

编辑neutron配置文件

> vi /etc/neutron/neutron.conf

```bash
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
transport_url = rabbit://openstack:asd@controller
auth_strategy = keystone
notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True

[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
[cors]
[cors.subdomain]

[database]
# connection = sqlite:////var/lib/neutron/neutron.sqlite
connection = mysql+pymysql://neutron:asd@controller/neutron

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = neutron
password = asd

[matchmaker_redis]

[nova]
auth_url = http://controller:35357
auth_type = password
project_domain_name = Default
user_domain_name = Default
region_name = RegionOne
project_name = service
username = nova
password = asd

[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[qos]
[quotas]
[ssl]
```

配置ml2
> vi /etc/neutron/plugins/ml2/ml2_conf.ini

```bash
[DEFAULT]
[ml2]
type_drivers = local,flat,vlan,gre,vxlan
tenant_network_types = vxlan
mechanism_drivers = openvswitch,l2population
extension_drivers = port_security
[ml2_type_flat]
flat_networks = provider
[ml2_type_geneve]
[ml2_type_gre]
tunnel_id_ranges = 500:1000
[ml2_type_vlan]
network_vlan_ranges = provider:2001:3000
[ml2_type_vxlan]
vni_ranges = 1:1000
[securitygroup]
enable_ipset = True
```

创建OVS桥接口

> ovs-vsctl add-br br-provider  
PS: `如果服务启动时没有匹配到接口，将中止进程`

配置openvswitch
> vi /etc/neutron/plugins/ml2/openvswitch_agent.ini
 
```
[DEFAULT]
[agent]
tunnel_types = vxlan,gre
l2_population = True
[ovs]
bridge_mappings = provider:br-provider
local_ip = 192.168.1.11
[securitygroup]
firewall_driver = iptables_hybrid
```
PS: `local_ip设置为用于overlay的控制节点接口IP`

配置L3代理
> vi /etc/neutron/l3_agent.ini
```
[DEFAULT]
interface_driver = openvswitch
external_network_bridge =
[AGENT]
```

配置DHCP代理
> vi /etc/neutron/dhcp_agent.ini

```
[DEFAULT]
interface_driver = openvswitch
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True
[AGENT]
```

---

配置metadata代理

> vi /etc/neutron/metadata_agent.ini
```
[DEFAULT]
nova_metadata_ip = controller
metadata_proxy_shared_secret = asd
[AGENT]
[cache]
```

配置nova使用neutron
> vi /etc/nova/nova.conf
```
...+
[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = Default
user_domain_name = Default
region_name = RegionOne
project_name = service
username = neutron
password = asd
service_metadata_proxy = True
metadata_proxy_shared_secret = asd
```

初始化neutron数据库
> su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron

重启nova-api服务
> service nova-api restart

重启neutron及OVS服务
> service neutron-server restart  
> service neutron-linuxbridge-agent restart  
> service neutron-dhcp-agent restart  
> service neutron-metadata-agent restart  
> service neutron-l3-agent restart  

> service openvswitch-switch restart  
> service neutron-openvswitch-agent restart  


验证操作
---

加载admin变量
>. admin-openrc

查看neutron扩展的运行情况
> neutron ext-list

查看网络组件运行情况
> openstack network agent list
```
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 3651645f-596f-4012-8b49-c6b2a9480d20 | DHCP agent         | controller | nova              | True  | UP    | neutron-dhcp-agent        |
| 991709fe-0de5-4a60-a00f-56c2f7fa5437 | Metadata agent     | controller | None              | True  | UP    | neutron-metadata-agent    |
| be6e09fe-7f86-47d2-a046-7f402e5f0336 | Open vSwitch agent | controller | None              | True  | UP    | neutron-openvswitch-agent |
| bffb2f7c-38cb-4589-b84c-94af0eed4862 | L3 agent           | controller | nova              | True  | UP    | neutron-l3-agent          |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
```

## Neutron OVS部署 - 计算节点

安装neutron linuxbridge代理
> apt install neutron-openvswitch-agent
```
[DEFAULT]
core_plugin = ml2
transport_url = rabbit://openstack:asd@controller
auth_strategy = keystone
[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf
[cors]
[cors.subdomain]
[database]
connection = sqlite:////var/lib/neutron/neutron.sqlite
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = neutron
password = asd
[matchmaker_redis]
[nova]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[qos]
[quotas]
[ssl]
```


vi /etc/neutron/plugins/ml2/openvswitch_agent.ini
```
[DEFAULT]
[agent]
tunnel_types = vxlan,gre
l2_population = True
[ovs]
local_ip = 192.168.1.21
[securitygroup]
```

> `local_ip设置为用于overlay的计算节点接口IP`

> vi /etc/nova/nova.conf
```
...+
[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = Default
user_domain_name = Default
region_name = RegionOne
project_name = service
username = neutron
password = asd
```
重启nova-compute服务
> service nova-compute restart

重启neutron及OVS服务
> service openvswitch-switch restart
> service neutron-openvswitch-agent restart

验证操作
---

在控制节点上进行验证操作

加载admin变量
. admin-openrc

查看网络组件运行情况

> openstack network agent list

```
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 3651645f-596f-4012-8b49-c6b2a9480d20 | DHCP agent         | controller | nova              | True  | UP    | neutron-dhcp-agent        |
| 991709fe-0de5-4a60-a00f-56c2f7fa5437 | Metadata agent     | controller | None              | True  | UP    | neutron-metadata-agent    |
| be6e09fe-7f86-47d2-a046-7f402e5f0336 | Open vSwitch agent | controller | None              | True  | UP    | neutron-openvswitch-agent |
| bffb2f7c-38cb-4589-b84c-94af0eed4862 | L3 agent           | controller | nova              | True  | UP    | neutron-l3-agent          |
| f4f0b972-fa78-4442-b18d-e946ffe70373 | Open vSwitch agent | compute    | None              | True  | UP    | neutron-openvswitch-agent |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
```

