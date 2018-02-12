## 部署LBaaS v2

安装neutron-lbaasv2软件包
> apt install neutron-lbaasv2-agent

编辑neutron_lbaas配置
> vi /etc/neutron/neutron_lbaas.conf
```bash
[DEFAULT]
[certificates]
[quotas]
[service_auth]
[service_providers]
service_provider = LOADBALANCERV2:Haproxy:neutron_lbaas.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
```

编辑lbaas_agent配置
> vi /etc/neutron/lbaas_agent.ini
```bash
[DEFAULT]
device_driver = neutron_lbaas.drivers.haproxy.namespace_driver.HaproxyNSDriver
interface_driver = openvswitch
[haproxy]
user_group = haproxy
```
> 根据实际部署的l2-agent选择接口驱动(openvswitch/linuxbridge)

更新neutron数据库
> neutron-db-manage --subproject neutron-lbaas upgrade head

运行lbaasv2
> neutron-lbaasv2-agent --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/lbaas_agent.ini

> PS: `经测试服务会自动运行，无需手动加载`

重启服务以生效
```bash
service neutron-server restart
service neutron-dhcp-agent restart
service neutron-metadata-agent restart
service openvswitch-switch restart
service neutron-openvswitch-agent restart
service neutron-l3-agent restart
```
