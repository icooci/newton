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


