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
本条命令没有回显

创建neutron服务实体
> openstack service create --name neutron --description "OpenStack Networking" network

创建neutron服务APT Endpoint
> openstack endpoint create --region RegionOne network public http://controller:9696
> openstack endpoint create --region RegionOne network internal http://controller:9696
> openstack endpoint create --region RegionOne network admin http://controller:9696
