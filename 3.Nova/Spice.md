
## Spice部署

控制节点配置
---

安装nova-spiceproxy
> apt install nova-spiceproxy spice-html5


配置nova
> vi /etc/nova/nova.conf 
```bash
[DEFAULT]
vnc_enabled = False

#[spice]
#enabled = True
#server_listen = $my_ip
#server_proxyclient_address = $my_ip

[spice]
enabled=True
html5proxy_base_url=http://192.168.1.11:6082/spice_auto.html
keymap=en-us
```

计算节点配置
---

> vi /etc/nova/nova.conf 
```bash
[DEFAULT]
vnc_enabled = False

#[spice]
#enabled = True
#html5proxy_base_url = http://controller:6082/spice_auto.html
#keymap = en-us
#server_listen = 0.0.0.0
#server_proxyclient_address = 192.168.1.21

[spice]
agent_enabled=True
enabled=True
html5proxy_base_url=http://192.168.1.11:6082/spice_auto.html
keymap=en-us
server_listen=0.0.0.0
server_proxyclient_address=192.168.1.21
```

