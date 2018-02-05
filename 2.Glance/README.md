Glance
---

创建glance数据库
> mysql -u root -p
```
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'asd';
```

加载admin变量
> . admin-openrc

创建glance用户
> openstack user create --domain default --password-prompt glance

为glance用户分配admin角色
> openstack role add --project service --user glance admin  
> `本条命令无回显`

创建服务实体
> openstack service create --name glance --description "OpenStack Image" image

创建glance服务API Endpoint
> openstack endpoint create --region RegionOne image public http://controller:9292  
> openstack endpoint create --region RegionOne image internal http://controller:9292  
> openstack endpoint create --region RegionOne image admin http://controller:9292  

安装glance软件包
> apt install glance

> vi /etc/glance/glance-api.conf 
```
[DEFAULT]
[cors]
[cors.subdomain]
[database]
sqlite_db = /var/lib/glance/glance.sqlite
backend = sqlalchemy
connection = mysql+pymysql://glance:asd@controller/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[image_format]
disk_formats = ami,ari,aki,vhd,vhdx,vmdk,raw,qcow2,vdi,iso,root-tar
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = asd
[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
[store_type_location_strategy]
[task]
[taskflow_executor]
```

> vi /etc/glance/glance-registry.conf
```
[DEFAULT]
[database]
sqlite_db = /var/lib/glance/glance.sqlite
backend = sqlalchemy
connection = mysql+pymysql://glance:asd@controller/glance
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = asd
[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone
[profiler]
```
asd

