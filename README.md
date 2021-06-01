### 一、安装ansible
```
yum -y install centos-release-ansible-29.noarch
yum -y install ansible
```



### 二、修改配置
- hosts
```
cat <<EOF > hosts
[all:vars]
ansible_user=root
ansible_password=root
#ansible_ssh_private_key_file=/root/qc.rsa
ansible_connection=paramiko

[rangers]
srv1 ansible_host=192.168.56.31
srv2 ansible_host=192.168.56.32
srv3 ansible_host=192.168.56.33
EOF
```
变量名 | 解释 | 备注
---|---|---
ansible_user | ansible ssh主机的用户 | sudo用户需要添加额外参数ansible_become=true
ansible_password | ansible ssh主机的用户的密码 |
srv1 ansible_host=192.168.56.31 | srv1将会设置为最终的主机名 |


- vars.yml
```
rangers_ssh_private_key: Deploy.key
ranger_user: { "user": "demo", "group": "demo", "password": "demoPWD" }
format_confirm: no
data_disks:
  - {"device": "/dev/sdb1", "mount_point": "/data00"}
    - {"device": "/dev/sdc1", "mount_point": "/data01"}
jumper_host: "{{ groups['rangers'][0] }}"
home_link: yes
home_link_dir: "{{ data_disks[0]['mount_point'] }}"
timezone: Asia/Shanghai
```
变量名 | 解释 | 备注
---|---|---
rangers_ssh_private_key | ssh密钥对私钥名 | 默认为Deploy.key
ranger_user | 设置sudo用户名、组、密码 |
format_confirm | bool值，是否格式化磁盘 | 需要格式化请修改为yes，请谨慎使用
data_disks | 数据盘和挂载点 |
jumper_host | 运行产品安装脚本的主机 | 默认为hosts列表中的第一台主机
home_link | bool值，是否创建sudo用户家目录软链接 |
home_link_dir | sudo用户软链接的目标目录 | 默认为第一块盘的挂载点
timezone | 时区 |



### 三、运行ansible脚本
```
ansible-playbook -i hosts rangers-preinstall.yml
```



### debug
```
ansible -i hosts all -m ping
ansible -i hosts all -m raw -a "free -g"
ansible -i hosts all -m service -a "name=chrony state=restarted"
```
