### 环境变量
```
vars:
```

### 主机列表 hosts
```
[all:vars]
ansible_user=root
ansible_password=root
#ansible_ssh_private_key_file=/srv/salt-minion/demo.rsa
ansible_connection=paramiko

```

### preinstall
```
ansible-playbook -i hosts rangers-preinstall.yml
```

### debug
```
ansible -i hosts all -m ping
ansible -i hosts all -m raw -a "free -g"
ansible -i hosts all -m service -a "name=chrony state=restarted"
```
