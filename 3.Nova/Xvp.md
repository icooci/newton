## xvpvnc部署

**控制节点**

安装ova-xvpvncproxy软件包	
> apt install nova-xvpvncproxy

**计算节点**

修改nova配置
> vi /etc/nova/nova.conf

```bash
[vnc]
..+
xvpvncproxy_base_url = http://controller:6081/console
```

Client
---

![image](https://github.com/icooci/newton/blob/master/3.Nova/xvp.png)

