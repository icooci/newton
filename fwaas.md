#### Firewall-as-a-Service v1

启用FWaaS插件

> vi /etc/neutron/neutron.conf
```
service_plugins = firewall

[service_providers]
service_provider = FIREWALL:Iptables:neutron.agent.linux.iptables_
firewall.OVSHybridIptablesFirewallDriver:default

[fwaas]
agent_version = v1
driver = iptables
enabled = True
```

> vi /etc/neutron/fwaas_driver.ini
```
[fwaas]
driver = neutron_fwaas.services.firewall.drivers.linux.iptables_
fwaas.IptablesFwaasDriver
enabled = True
```

配置L3扩展fwaas

> vi /etc/neutron/l3_agent.ini
```
[AGENT]
extensions = fwaas
```

创建fwaas表

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
