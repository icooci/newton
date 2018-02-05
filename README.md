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
创建keystone数据库
> mysql -u root -p  
```
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'asd';
```
安装keystone软件包  
> apt install keystone  

编辑keystone配置文件  
> vi /etc/keystone/keystone.conf  

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
初始化keystone数据库  

> keystone-manage db_sync  

初始化Fernet  

> keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone  
> keystone-manage credential_setup --keystone-user keystone --keystone-group keystone  

引导身份认证服务  

> keystone-manage bootstrap --bootstrap-password asd \
  --bootstrap-admin-url http://controller:35357/v3/ \
  --bootstrap-internal-url http://controller:35357/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne

配置apache  

> vi /etc/apache2/apache2.conf
```diff
+ ServerName controller
```

> service apache2 restart  

删除默认SQLite数据库  

> rm -f /var/lib/keystone/keystone.db

配置管理员账户
```bash
export OS_USERNAME=admin
export OS_PASSWORD=asd
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
```
创建service项目，用于每个服务

> openstack project create --domain default --description "Service Project" service


创建demo项目，用于一般权限任务

> openstack project create --domain default --description "Demo Project" demo

创建demo用户

> openstack user create --domain default --password-prompt demo

创建user角色

> openstack role create user

赋予demo用户user角色

> openstack role add --project demo --user demo user  
`本条命令无回显`
