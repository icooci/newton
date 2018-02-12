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

配置L3代理
vi /etc/neutron/l3_agent.ini
```
[DEFAULT]
interface_driver = openvswitch
external_network_bridge =
[AGENT]
```

配置DHCP代理
vi /etc/neutron/dhcp_agent.ini

```
[DEFAULT]
interface_driver = openvswitch
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True
[AGENT]
```
