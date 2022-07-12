# Tidb安装部署

## 前期准备：
### 关闭swap
``` bash 
#echo "vm.swappiness = 0">> /etc/sysctl.conf
#一起执行 swapoff -a 和 swapon -a 命令是为了刷新 swap，将 swap 里的数据转储回内存，并清空 swap 里的数据。不可省略 swappiness 设置而只执行 swapoff -a；否则，重启后 swap 会再次自动打开，使得操作失效。
#执行 sysctl -p 命令是为了在不重启的情况下使配置生效。
#swapoff -a && swapon -a
#sysctl -p
```
### 关闭防火墙
``` bash 
#systemctl stop firewalld.service
#systemctl disable firewalld.service
```
### 安装ntp服务
``` bash 
#yum install -y ntp
#systemctl start ntpd
#systemctl enable ntpd
```
### 检查和配置操作系统优化参数
- 关闭透明大页（即 Transparent Huge Pages，缩写为 THP）。
  - 数据库的内存访问模式往往是稀疏的而非连续的。当高阶内存碎片化比较严重时，分配 THP 页面会出现较高的延迟。
  - 执行以下命令查看透明大页的开启状态。
  ``` bash 
  #cat /sys/kernel/mm/transparent_hugepage/enabled
  always madvise [never] 
  #表示透明大页处于启用状态，需要关闭
  ```
  
- 将存储介质的 I/O 调度器设置为 noop。对于高速 SSD 存储介质，内核的 I/O 调度操作会导致性能损失。将调度器设置为 noop 后，内核不做任何操作，直接将 I/O 请求下发给硬件，以获取更好的性能。同时，noop 调度器也有较好的普适性。
  - 查看数据目录所在磁盘的 I/O 调度器 
  ``` bash 
  #cat /sys/block/sda/queue/scheduler
  noop [deadline] cfq
  noop [deadline] cfq
  #noop [deadline] cfq 表示磁盘的 I/O 调度器使用 deadline，需要进行修改
  ``` 
  - 执行以下命令查看磁盘的唯一标识 ID_SERIAL。
  ``` bash 
  #udevadm info --name=/dev/sdb | grep ID_SERIAL
  E: ID_SERIAL=36d0946606d79f90025f3e09a0c1f9e81
  E: ID_SERIAL_SHORT=6d0946606d79f90025f3e09a0c1f9e81
  ``` 

- 为调整 CPU 频率的 cpufreq 模块选用 performance 模式。将 CPU 频率固定在其支持的最高运行频率上，不进行动态调节，可获取最佳的性能。
  - 执行以下命令查看 cpufreq 模块选用的节能策略。
  ``` bash 
  #cpupower frequency-info --policy
  analyzing CPU 0:
  current policy: frequency should be within 1.20 GHz and 3.10 GHz.
  The governor "powersave" may decide which speed to use within this range.
  #The governor "powersave" 表示 cpufreq 的节能策略使用 powersave，需要调整为 performance 策略。如果是虚拟机或者云主机，则不需要调整，命令输出通常为 Unable to determine current policy。
  ```
- 通过tuned修改以上参数
``` bash 
#mkdir /etc/tuned/balanced-tidb-optimal/
#vi /etc/tuned/balanced-tidb-optimal/tuned.conf
[main]
include=balanced

[cpu]
governor=performance

[vm]
transparent_hugepages=never

[disk]
devices_udev_regex=(ID_SERIAL=36d0946606d79f90025f3e09a0c1fc035)|(ID_SERIAL=36d0946606d79f90025f3e09a0c1f9e81)
elevator=noop

#tuned-adm profile balanced-tidb-optimal
```

## 使用 TiUP 部署 TiDB 集群
## 使用tiup部署 
- 下载tiup并配置环境
``` bash 
#curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh

#声明环境变量
#source source .bash_profile

#安装tiup cluster组件 
#tiup cluster

#如果已经安装，则更新 TiUP cluster 组件至最新版本：
#tiup update --self && tiup update cluster

#验证当前 TiUP cluster 版本信息。执行如下命令查看 TiUP cluster 组件版本：
#tiup --binary cluster
```

- 初始化集群拓扑文件
``` bash 
#生成默认的单集群配置文件，该文件用于构建集群使用。
#tiup cluster template --full > topology.yaml
#生成库区域的集群配置文件
#tiup cluster template --multi-dc > topology.yaml

#以下为单集群的配置文件模板
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/usr/local/tidb-deploy"
  data_dir: "/data/tidb-data"
  arch: "amd64"
monitored:
  node_exporter_port: 9100
  blackbox_exporter_port: 9115
pd_servers:
  - host: 10.105.21.183
  - host: 10.105.21.182
  - host: 10.105.21.181
tidb_servers:
  - host: 10.105.21.183
  - host: 10.105.21.182
  - host: 10.105.21.181
tikv_servers:
  - host: 10.105.21.183
  - host: 10.105.21.182
  - host: 10.105.21.181
tiflash_servers:
  - host: 10.105.21.182
  - host: 10.105.21.181
monitoring_servers:
  - host: 10.105.21.182
grafana_servers:
  - host: 10.105.21.181
alertmanager_servers:
  - host: 10.105.21.181

```

- 配置所有节点的免密钥直接登陆 

- 部署tidb
``` bash 
#检查集群安装存在的潜在风险，主要是验证部署中一些依赖包是否安装，还有一些步骤是否能执行：
#tiup cluster check ./topology.yaml --user root

#自动修复集群存在的潜在风险：
#tiup cluster check ./topology.yaml --apply --user root

#部署tidb
#tiup cluster deploy tidb-test v6.1.0 ./topology.yaml --user root

#tidb-test 为部署的集群名称，可任意更改。
#v6.1.0 为部署的集群版本，可以通过执行 tiup list tidb 来查看 TiUP 支持的最新可用版本。
#初始化配置文件为 topology.yaml。
#--user root 表示通过 root 用户登录到目标主机完成集群部署，该用户需要有 ssh 到目标机器的权限，并且在目标机器有 sudo 权限。也可以用其他有 ssh 和 sudo 权限的用户完成部署。
#[-i] 及 [-p] 为可选项，如果已经配置免密登录目标机，则不需填写。否则选择其一即可，[-i] 为可登录到目标机的 root 用户（或 --user 指定的其他用户）的私钥，也可使用 [-p] 交互式输入该用户的密码。
```

- 查看 TiUP 管理的集群情况
``` bash 
#TiUP 支持管理多个 TiDB 集群，该命令会输出当前通过 TiUP cluster 管理的所有集群信息，包括集群名称、部署用户、版本、密钥信息等。
#tiup cluster list

#检查 tidb-test 集群情况：
#tiup cluster display tidb-test
``` 

### 启动集群
``` bash
#方式一：安全启动,安全启动会自动生成root密码，需要使用密码才能登陆
#tiup cluster start tidb-test --init

#方式二：普通启动 使用普通启动方式后，可通过无密码的 root 用户登录数据库。
#tiup cluster start tidb-test

#验证集群运行状态
#tiup cluster display tidb-test
```

