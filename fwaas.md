## Firewall-as-a-Service v1


启用FWaaS插件

> vi /etc/neutron/neutron.conf
```
service_plugins = firewall

[service_providers]
service_provider = FIREWALL:Iptables:neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver:default

[fwaas]
agent_version = v1
driver = iptables
enabled = True
```

>目前测试不配置service_providers依然能正常运作

> vi /etc/neutron/fwaas_driver.ini
```
[fwaas]
driver = neutron_fwaas.services.firewall.drivers.linux.iptables_fwaas.IptablesFwaasDriver
enabled = True
```

配置L3扩展fwaas

> vi /etc/neutron/l3_agent.ini
```
[AGENT]
extensions = fwaas
```

更新数据库

> neutron-db-manage --subproject neutron-fwaas upgrade head


开启图形界面 (默认为开启)

> vi /etc/openstack-dashboard/local_settings.py
```
OPENSTACK_NEUTRON_NETWORK = {
    ...
    'enable_firewall' = True,
    ...
}
```

重启服务以生效
> service neutron-server restart  
> service neutron-l3-agent restart
