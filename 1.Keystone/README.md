## Keystone部署

创建keystone数据库
> mysql -u root -p  
```
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'asd';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'asd';
exit;
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

> su -s /bin/sh -c "keystone-manage db_sync" keystone  
> or `keystone-manage db_sync`

初始化Fernet  

> keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone  
> keystone-manage credential_setup --keystone-user keystone --keystone-group keystone  

引导身份认证服务  

```bash
keystone-manage bootstrap --bootstrap-password asd \
  --bootstrap-admin-url http://controller:35357/v3/ \
  --bootstrap-internal-url http://controller:35357/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

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

关闭临时认证机制  

> vi /etc/keystone/keystone-paste.ini
```diff
- admin_token_auth
```
`分别从[pipeline:public_api], [pipeline:admin_api]和[pipeline:api_v3]部分删除 admin_token_auth `

取消临时变量  

> unset OS_AUTH_URL OS_PASSWORD  


验证操作
---


1. 使用admin用户请求token  
```bash
openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name Default \
  --os-user-domain-name Default \
  --os-project-name admin \
  --os-username admin \
  token issue
```
服务器返回结果如下:
```
+------------+-------------------------------------------------------------------------------------+
| Field      | Value                                                                               |
+------------+-------------------------------------------------------------------------------------+
| expires    | 2018-02-05 05:12:37+00:00                                                           |
| id         | gAAAAABad9m1ndMN48wZoj2flMPvOXLFo4crVKJe48xiZcqEE_-BOx3e7F7GdHl8Noxamu3cA_AavM3JQqe |
|            | xaSzpumywkOfwy1kZoIqX5WVmK0H7aGJYvvGEDXKsY829wxvPL0L0Us3nAuShkJrS0DRiiDUzWNIyjElQiV |
|            | CDjHXjITJ1D0L3-Rg                                                                   |
| project_id | 504960eb27594515a5c52299e592bdb2                                                    |
| user_id    | 5c30414ba4f14019ba86e1b5a3985856                                                    |
+------------+-------------------------------------------------------------------------------------+
```

2. 使用demo用户请求token  

```
openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name Default \
  --os-user-domain-name Default \
  --os-project-name demo \
  --os-username demo \
  token issue
```
服务器返回结果如下:
```
+------------+-------------------------------------------------------------------------------------+
| Field      | Value                                                                               |
+------------+-------------------------------------------------------------------------------------+
| expires    | 2018-02-05 05:13:56+00:00                                                           |
| id         | gAAAAABad9oE9OyDhsFj3hPsGy1yht5cIP-                                                 |
|            | m65JP3xHiwiJ5_8bVFZAPwsdk3X9zrrFYdcX5KWmbDHzizihPrRU_YAeku86ZPjZKeQlU3dNgpKznC-     |
|            | XT0ID2LUpLLfQYby15TVfOEO5rz7YYhrayvErAmoYDelRWJXn9ua3r9WyGJjLF42R2_gc               |
| project_id | 51453ea6df854c6b807ddbfe214bd322                                                    |
| user_id    | cc3d529c69e34b219270a3145624ae5c                                                    |
+------------+-------------------------------------------------------------------------------------+
```

配置环境变量脚本
> vi admin-openrc
```bash
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=asd
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

> vi demo-openrc
```bash
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=asd
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

加载admin环境变量  

> . admin-openrc  

申请token  

> openstack token issue  

服务器返回结果如下:
```
+------------+-------------------------------------------------------------------------------------+
| Field      | Value                                                                               |
+------------+-------------------------------------------------------------------------------------+
| expires    | 2018-02-05 05:52:33+00:00                                                           |
| id         | gAAAAABad-MRnDoOXPyhHYENp7XauDPFCF4wxR_EjpIDN2Wc2veywDNsFx3tH1vNWH7hE14ti-bRLA23GoT |
|            | zPyOzLvKpxxVoySGFKdIxrSSUQs05k0mbLwPt1RrpUCIXldALu80ZteznfKUyvm12pPyuJ4qw1RxvpT_SJT |
|            | Y3A-XOP5QhB2t_dh8                                                                   |
| project_id | 504960eb27594515a5c52299e592bdb2                                                    |
| user_id    | 5c30414ba4f14019ba86e1b5a3985856                                                    |
+------------+-------------------------------------------------------------------------------------+
```
