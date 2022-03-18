### 说明
脚本仅用于minibase部署前的准备工作，具体会做如下事情
1. 设置时区
2. 设置主机名
3. 修改/etc/hosts
4. 磁盘分区、格式化、挂载(带/etc/fstab)
5. 创建sudo用户(minibase)并设置密码，并生成ssh密钥对(Deploy.key)
7. 软连接sudo用户家目录到数据目录/data00/minibase  

注： 
- 执行该脚本的主机可以是任一主机；
- 除格式化磁盘外，可多次执行，最终结果具有一致性；
- 磁盘格式化完成后，若要再次执行格式化，需要先进行磁盘的umount操作




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

[minibase]
minibase-srv1 ansible_host=192.168.56.31
minibase-srv2 ansible_host=192.168.56.32
minibase-srv3 ansible_host=192.168.56.33
EOF
```
变量名 | 解释 | 备注
---|---|---
ansible_user| 用于ssh连接主机的用户
ansible_password| 用于ssh连接主机的密码| 适用于密码登录场景
ansible_ssh_private_key_file| 用于ssh连接主机的私钥| 适用于密钥登录场景
minibase-srv1 ansible_host=192.168.56.31| minibase-srv1为主机名

> 有需要可以指定python路径ansible_python_interpreter=python3


- vars.yml
```
minibase_ssh_private_key: Deploy.key
ranger_user: { "user": "demo", "group": "demo", "password": "demoPWD", "uid": "1040" }
format_confirm: "{{ format_force | default('no') }}"
data_disks:
  - {"device": "/dev/sdb", "mount_point": "/data00"}
  - {"device": "/dev/sdc", "mount_point": "/data01"}
home_link: yes
home_link_dir: "{{ data_disks[0]['mount_point'] }}"
jumper_host: "{{ groups['minibase'][0] }}"
timezone: Asia/Shanghai
```
变量名 | 解释 | 备注
---|---|---
minibase_ssh_private_key| ssh密钥对私钥名| 默认Deploy.key|
ranger_user| 设置sudo用户、组、密码| 默认dataminibase|
format_confirm| bool值，是否格式化磁盘| 默认为no，可通过自定义参数format_force修改|
data_disks| 数据盘和挂载点| |
jumper_host| 运行产品安装脚本的主机| 默认为hosts列表中的第一台主机|
home_link| bool值，是否创建sudo用户家目录的软链接| 用于dataminibase 家目录过小|
home_link_dir| sudo用户软链接的目标目录| 默认为第一块盘的挂载点/data00/dataminibase|
timezone| 时区| 默认Asia/Shanghai|



### 三、运行ansible脚本
```
ansible-playbook -i hosts minibase-preinstall.yml
```
```
#可通过标签指定需要的task
ansible-playbook -i hosts minibase-preinstall.yml -t timezone,hostname,hosts
```
共包含如下标签  
- timezone
- hostname
- hosts
- disk
- user
- home_link

```
#强制格式化磁盘，慎用
#ansible-playbook -i hosts minibase-preinstall.yml -e format_force=yes
```


### debug
```
ansible -i hosts all -m ping
ansible -i hosts all -m raw -a "free -g"
ansible -i hosts all -m service -a "name=chrony state=restarted"
```



### 四、新增节点
- hosts
```
cat <<EOF > hosts
[all:vars]
ansible_user=sudoer_user
#ansible_password=sudoer_pwd
ansible_ssh_private_key_file=/tmp/sudoer_private_key.rsa
ansible_become=true
ansible_connection=paramiko

[minibase]
minibase-srv1 ansible_host=192.168.56.31
minibase-srv2 ansible_host=192.168.56.32
minibase-srv3 ansible_host=192.168.56.33

#增加[add]章节
[add]
minibase-srv4 ansible_host=192.168.56.34
minibase-srv5 ansible_host=192.168.56.35
minibase-srv6 ansible_host=192.168.56.36
EOF
```
- 执行playbook
```
#注: -l add 指定章节一定不要漏不能错
ansible-playbook -i hosts minibase-preinstall.yml -l add
```
