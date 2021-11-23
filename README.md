# Kubernetes v1.20 企业级高可用集群自动部署（离线版）
>### 注：确保所有节点系统时间一致
>### 操作系统要求：CentOS7.x_x64
>### 2021-11-23
>### Jerry.jin

### 1、找一台服务器安装Ansible
```
# yum install epel-release -y
# yum install ansible -y
```
### 2、下载所需文件

下载Ansible部署文件：

```
# git clone https://github.com/jinyan2049/auto-install-k8s
# cd auto-install-k8s
```

下载准备好软件包（包含所有涉及文件和镜像，比较大），解压到/root目录：

云盘链接：https://pan.baidu.com/s/1uCLylsj1-W2HigS_Tn9b5g 
提取码：bicc 
```
# tar zxf binary_pkg.tar.gz
```
### 3、修改Ansible文件

修改hosts文件，根据规划修改对应IP和名称。

```
# vi hosts
...
```
修改group_vars/all.yml文件，修改软件包目录和证书可信任IP。

```
# vim group_vars/all.yml
software_dir: '/root/binary_pkg'
...
cert_hosts:
  k8s:
  etcd:
```
## 4、一键部署
### 4.1 架构图
单Master架构
![avatar](https://github.com/jinyan2049/auto-install-k8s/blob/master/single-master.jpg)

多Master架构
![avatar](https://github.com/jinyan2049/auto-install-k8s/blob/master/multi-master.jpg)
### 4.2 部署命令
单Master版：
```
# ansible-playbook -i hosts single-master-deploy.yml -uroot -k
```
多Master版：
```
# ansible-playbook -i hosts multi-master-deploy.yml -uroot -k
```

## 5、查看集群节点
```
# kubectl get node
NAME          STATUS   ROLES    AGE   VERSION
k8s-master    Ready    <none>   9h    v1.20.4
k8s-node1     Ready    <none>   9h    v1.20.4
k8s-node2     Ready    <none>   9h    v1.20.4
```

## 6、其他
### 6.1 部署控制
如果安装某个阶段失败，可针对性测试.

例如：只运行部署插件
```
# ansible-playbook -i hosts single-master-deploy.yml -uroot -k --tags addons
```

### 6.2 节点扩容
1）修改hosts，添加新节点ip
```
# vi hosts
...
[newnode]
192.168.31.75 node_name=k8s-node3
```
2）执行部署
```
# ansible-playbook -i hosts add-node.yml -uroot -k
```
### 6.3 所有HTTPS证书存放路径
部署产生的证书都会存放到目录“auto-install-k8s-master/ssl”，一定要保存好，后面还会用到~

### 6.4 忘记token解决办法
kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')

## 7 安装bash-completion
yum install bash-completion

### 7.1 马上生效
source /usr/share/bash-completion/bash_completion

source <(kubectl completion bash)

### 7.2 添加环境变量
echo "source <(kubectl completion bash)" >> ~/.bashrc

### 7.3 通过命令行验证设置是否正常
type _init_completion

### 8 删除node
kubectl drain 节点名字 --delete-local-data --force --ignore-daemonsets

kubectl delete node 节点名字
<br/>
<br/>


