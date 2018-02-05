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


