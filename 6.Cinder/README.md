## Cinder部署 - 控制节点

创建cinder数据库

>  mysql -u root -p
```
CREATE DATABASE cinder;

GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY 'asd';

EXIT;
```
加载admin变量
> . admin-openrc

创建cinder用户
> openstack user create --domain default --password-prompt cinder

为cinder用户分配admin角色
> openstack role add --project service --user cinder admin  
> `本条命令无回显`

创建cinder/cinderv2服务实体
> openstack service create --name cinder --description "OpenStack Block Storage" volume  
> openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2

创建cinder服务APT Endpoint
> openstack endpoint create --region RegionOne volume public http://controller:8776/v1/%\(tenant_id\)s  
> openstack endpoint create --region RegionOne volume internal http://controller:8776/v1/%\(tenant_id\)s  
> openstack endpoint create --region RegionOne volume admin http://controller:8776/v1/%\(tenant_id\)s  

创建cinderv2服务API Endpoint
> openstack endpoint create --region RegionOne volumev2 public http://controller:8776/v2/%\(tenant_id\)s  
> openstack endpoint create --region RegionOne volumev2 internal http://controller:8776/v2/%\(tenant_id\)s  
> openstack endpoint create --region RegionOne volumev2 admin http://controller:8776/v2/%\(tenant_id\)s  

安装cinder软件包
> apt install cinder-api cinder-scheduler

编辑cinder配置
> vi /etc/cinder/cinder.conf
```bash
[DEFAULT]
rootwrap_config = /etc/cinder/rootwrap.conf
api_paste_confg = /etc/cinder/api-paste.ini
iscsi_helper = tgtadm
volume_name_template = volume-%s
volume_group = cinder-volumes
verbose = True
auth_strategy = keystone
state_path = /var/lib/cinder
lock_path = /var/lock/cinder
volumes_dir = /var/lib/cinder/volumes
transport_url = rabbit://openstack:asd@controller
my_ip = 192.168.1.11

[database]
connection = mysql+pymysql://cinder:asd@controller/cinder

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = cinder
password = asd

[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```
> `$my_ip 为控制节点管理IP`

添加nova配置
>vi /etc/nova/nova.conf
```bash
..+
[cinder]
os_region_name = RegionOne
```

重启nova-api服务

> service nova-api restart

重启cinder服务

> service cinder-scheduler restart  
> service cinder-api restart  

<br />
<br />

## Cinder部署 - 存储节点

安装LVM软件包(大部分发行版已包含)
> apt install lvm2

创建 Physical Volume
> pvcreate /dev/sda

创建 Volume Group 
> vgcreate cinder-volumes /dev/sda

配置LVM过滤器
> vi /etc/lvm/lvm.conf
```bash
devices {
...
filter = [ "a/sda/", "r/.*/"]
```
> 只允许/dev/sda，过滤其他所有dev  
> `PS: 如果OS所在磁盘上使用了LVM，则同样必须在此添加`  

安装cinder-volume软件包
> apt install cinder-volume

编辑cinder配置
> vi /etc/cinder/cinder.conf

```bash
[DEFAULT]
rootwrap_config = /etc/cinder/rootwrap.conf
api_paste_confg = /etc/cinder/api-paste.ini
iscsi_helper = tgtadm
volume_name_template = volume-%s
volume_group = cinder-volumes
verbose = True
auth_strategy = keystone
state_path = /var/lib/cinder
lock_path = /var/lock/cinder
volumes_dir = /var/lib/cinder/volumes
transport_url = rabbit://openstack:asd@controller
my_ip = 192.168.1.31
enabled_backends = lvm
glance_api_servers = http://controller:9292

[database]
connection = mysql+pymysql://cinder:asd@controller/cinder

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = cinder
password = asd

[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = tgtadm

[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```

> `$my_ip 为存储节点管理IP`

重启块存储服务
> service tgt restart  
> service cinder-volume restart  
