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
ansible_user=sudoer_user
#ansible_password=sudoer_pwd
ansible_ssh_private_key_file=/tmp/sudoer_private_key.rsa
ansible_become=true
ansible_connection=paramiko

[rangers]
rangers-srv1 ansible_host=192.168.56.31
rangers-srv2 ansible_host=192.168.56.32
rangers-srv3 ansible_host=192.168.56.33
EOF
```
变量名 | 解释 | 备注
---|---|---
ansible_user| 用于ssh连接主机的用户
ansible_password| 用于ssh连接主机的密码| 适用于密码登录场景
ansible_ssh_private_key_file| 用于ssh连接主机的私钥| 适用于密钥登录场景
rangers-srv1 ansible_host=192.168.56.31| rangers-srv1为主机名

> 有需要可以指定python路径ansible_python_interpreter=python3


- vars.yml
```
rangers_ssh_private_key: Deploy.key
ranger_user: { "user": "demo", "group": "demo", "password": "demoPWD" }
format_confirm: no
data_disks:
  - {"device": "/dev/sdb1", "mount_point": "/data00"}
  - {"device": "/dev/sdc1", "mount_point": "/data01"}
home_link: yes
home_link_dir: "{{ data_disks[0]['mount_point'] }}"
jumper_host: "{{ groups['rangers'][0] }}"
timezone: Asia/Shanghai
```
变量名 | 解释 | 备注
---|---|---
rangers_ssh_private_key| ssh密钥对私钥名| 默认Deploy.key|
ranger_user| 设置sudo用户、组、密码| 默认datarangers|
format_confirm| bool值，是否格式化磁盘| 需要格式化请修改为yes，请谨慎使用|
data_disks| 数据盘和挂载点| |
jumper_host| 运行产品安装脚本的主机| 默认为hosts列表中的第一台主机|
home_link| bool值，是否创建sudo用户家目录的软链接| 用于datarangers 家目录过小|
home_link_dir| sudo用户软链接的目标目录| 默认为第一块盘的挂载点/data00/datarangers|
timezone| 时区| 默认Asia/Shanghai|



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
