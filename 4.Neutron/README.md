Neutron部署
---
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

安装neutron软件包
> apt install neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent

> vi /etc/neutron/neutron.conf
```
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
