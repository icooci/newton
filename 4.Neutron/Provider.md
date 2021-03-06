## Neutron部署 - 控制节点

创建neutron数据库
> mysql -u root -p
```
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'asd';
^D
```
加载admin变量
> . admin-openrc

创建neutron用户
> openstack user create --domain default --password-prompt neutron

为neutron用户分配admin角色
> openstack role add --project service --user neutron admin  
`本条命令无回显`

创建neutron服务实体
> openstack service create --name neutron --description "OpenStack Networking" network

创建neutron服务APT Endpoint
> openstack endpoint create --region RegionOne network public http://controller:9696  
> openstack endpoint create --region RegionOne network internal http://controller:9696  
> openstack endpoint create --region RegionOne network admin http://controller:9696  

<br />

网络类型: Provider网络配置
---

安装neutron软件包
> apt install neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent

编辑neutron配置文件
> vi /etc/neutron/neutron.conf

```bash
[DEFAULT]
core_plugin = ml2
service_plugins =
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
type_drivers = flat,vlan
tenant_network_types =
mechanism_drivers = linuxbridge
extension_drivers = port_security
[ml2_type_flat]
flat_networks = provider
[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
[ml2_type_vxlan]
[securitygroup]
enable_ipset = True
```

配置linuxbridge
> vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini

```bash
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:ens3
[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = False
```

配置dhcp代理
> vi /etc/neutron/dhcp_agent.ini

```bash
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True
[AGENT]
```
<br />

---

配置metadata代理
> vi /etc/neutron/metadata_agent.ini

```bash
[DEFAULT]
nova_metadata_ip = controller
metadata_proxy_shared_secret = asd
[AGENT]
[cache]
```

配置nova，添加neutron配置

> vi /etc/nova/nova.conf 

```bash
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

重启neutron服务
> service neutron-server restart  
> service neutron-linuxbridge-agent restart  
> service neutron-dhcp-agent restart  
> service neutron-metadata-agent restart  

重启l3代理服务(视网络类型而定)
>  service neutron-l3-agent restart

<br />

验证操作
---

加载admin变量
> . admin-openrc

查看neutron扩展的运行情况
> neutron ext-list

查看网络组件运行情况
> openstack network agent list

```
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 0b38231e-216f-4d62-97cd-423943131eaf | Metadata agent     | controller | None              | True  | UP    | neutron-metadata-agent    |
| 73b8299c-9073-42d0-b13d-8986513d15b1 | Linux bridge agent | controller | None              | True  | UP    | neutron-linuxbridge-agent |
| d34eb018-be45-4428-9afc-f9aeb561ee73 | DHCP agent         | controller | nova              | True  | UP    | neutron-dhcp-agent        |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
```
<br />
<br />

## Neutron部署 - 计算节点

安装neutron linuxbridge代理

> apt install neutron-linuxbridge-agent

配置neutron
> vi /etc/neutron/neutron.conf

```bash
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

配置linuxbridge
> vi /etc/neutron/plugins/ml2/linuxbridge_agent.ini

```bash
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:ens3
[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
[vxlan]
enable_vxlan = False
```

配置nova增加neutron配置
> vi /etc/nova/nova.conf

```bash
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

重启neutron服务
> service neutron-linuxbridge-agent restart

 
验证操作
---

在控制节点上进行验证操作

加载admin变量
> . admin-openrc

查看网络组件运行情况
> openstack network agent list

```
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 0b38231e-216f-4d62-97cd-423943131eaf | Metadata agent     | controller | None              | True  | UP    | neutron-metadata-agent    |
| 73b8299c-9073-42d0-b13d-8986513d15b1 | Linux bridge agent | controller | None              | True  | UP    | neutron-linuxbridge-agent |
| d34eb018-be45-4428-9afc-f9aeb561ee73 | DHCP agent         | controller | nova              | True  | UP    | neutron-dhcp-agent        |
| fd78fe29-841d-4c26-83b3-55b7272084bf | Linux bridge agent | compute    | None              | True  | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
```
