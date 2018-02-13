## NFS Volume Provider

安装NFS软件
> apt install nfs-common

在cinder中添加NFS配置
> vi /etc/cinder/cinder.conf
```
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
enabled_backends = lvm,nfs
glance_api_servers = http://controller:9292
debug = True

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
volume_backend_name = lvmbackend

[nfs]
nfs_shares_config = /etc/cinder/nfs_shares
volume_driver = cinder.volume.drivers.nfs.NfsDriver
volume_backend_name = nfsbackend

[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```

配置NFS连接信息
> vi /etc/cinder/nfs_shares 
`192.168.1.x:/zz`



