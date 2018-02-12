## Cinder部署

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

