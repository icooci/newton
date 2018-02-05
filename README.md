# Newton安装部署
1.环境准备
------
OS: Ubuntu Server 16.04 LTS  

| 节点 | 主机名 | 地址规划 |
| ---- | ----- | ------- |
| Controller Node | controller | 192.168.1.11 |
| Compute Node | compute | 192.168.1.21 |
| Block Storage Node | block | 192.168.1.31 |
| Object Storage Node 1 | object1 | 192.168.1.41 |
| Object Storage Node 2 | object2 | 192.168.1.42 |

网络配置  
> vi /etc/network/interfaces  
```
auto ens3
iface ens3 inet static
        address 192.168.1.11
        netmask 255.255.255.0
        network 192.168.1.0
        broadcast 192.168.1.255
        gateway 192.168.1.1
        dns-nameservers 192.168.1.1
```

网络名称解析  
vi /etc/hosts
```
# controller
192.168.1.11       controller

# compute
192.168.1.21       compute

# block
192.168.1.31       block

# object1
192.168.1.41       object1

# object2
192.168.1.42       object2
```
**控制节点部署NTP服务**  
> apt install chrony  
> vi /etc/chrony/chrony.conf  
```diff
+ allow 192.168.1.0/24
```
> service chrony restart  

**其他节点部署NTP服务**  
> apt install chrony  
> vi /etc/chrony/chrony.conf
```diff
- pool 2.debian.pool.ntp.org offline iburst
+ server controller iburst
```
> service chrony restart  

验证NTP同步信息  
> chronyc sources

**所有节点安装openstack client**  
> apt install software-properties-common

> add-apt-repository cloud-archive:newton  
[ENTER]

更新软件包  

> apt-get update && apt-get dist-upgrade

重启以应用新kernel  

> reboot

安装openstackclient  

> apt install python-openstackclient

**控制节点安装SQL**
>apt install mariadb-server python-pymysql

> vi /etc/mysql/mariadb.conf.d/99-openstack.cnf
```
[mysqld]
bind-address = 192.168.1.11

default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```
> service mysql restart

> mysql_secure_installation

**控制节点安装消息队列**  

> apt install rabbitmq-server

> rabbitmqctl add_user openstack asd

> rabbitmqctl set_permissions openstack ".*" ".*" ".*"

**控制节点安装Memcached**

> apt install memcached python-memcache

> vi /etc/memcached.conf
```diff
- -l 127.0.0.1
+ -l 192.168.1.11
```
> service memcached restart

2.认证服务
---
> mysql -u root -p  
```
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'asd';
```
> apt install keystone  

> vi keystone.conf  

```bash
[DEFAULT]
log_dir = /var/log/keystone
[assignment]
[auth]
[cache]
[catalog]
[cors]
[cors.subdomain]
[credential]
[database]
# connection = sqlite:////var/lib/keystone/keystone.db
connection = mysql+pymysql://keystone:asd@controller/keystone
[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[federation]
[fernet_tokens]
[identity]
[identity_mapping]
[kvs]
[ldap]
[matchmaker_redis]
[memcache]
[oauth1]
[os_inherit]
[oslo_messaging_amqp]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
[policy]
[profiler]
[resource]
[revoke]
[role]
[saml]
[security_compliance]
[shadow_users]
[signing]
[token]
provider = fernet
[tokenless_auth]
[trust]
[extra_headers]
Distribution = Ubuntu
```

> keystone-manage db_sync

> keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
> keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

> keystone-manage bootstrap --bootstrap-password asd \
  --bootstrap-admin-url http://controller:35357/v3/ \
  --bootstrap-internal-url http://controller:35357/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
