# tiup使用教程

- TiDB 提供了丰富的工具，可以帮助你进行部署运维、数据管理（例如，数据迁移、备份恢复、数据校验）、在 TiKV 上运行 Spark SQL。

## 下载tiup并配置环境
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

## tiup基础组件使用 
``` text
TiUP 主要通过以下一些命令来管理组件：
list：查询组件列表，用于了解可以安装哪些组件，以及这些组件可选哪些版本
install：安装某个组件的特定版本
update：升级某个组件到最新的版本
uninstall：卸载组件
status：查看组件运行状态
clean：清理组件实例
help：打印帮助信息，后面跟其他 TiUP 命令则是打印该命令的使用方法
```


###  tiup install 安装组件
    - tiup install <component>：安装指定组件的最新稳定版
    - tiup install <component>:[version]：安装指定组件的指定版本
``` text
示例一：使用 TiUP 安装最新稳定版的 TiDB
tiup install tidb

示例二：使用 TiUP 安装 nightly 版本的 TiDB
tiup install tidb:nightly

示例三：使用 TiUP 安装 v5.1.4 版本的 TiKV
tiup install tikv:v5.1.4
```
###  tiup uninstall 卸载组件
###  tiup update  tiup update 命令来升级组件。除了以下几个参数，该命令的用法基本和 tiup install 相同：
   - --all：升级所有组件
   - --nightly：升级至 nightly 版本
   - --self：升级 TiUP 自己至最新版本
   - --force：强制升级至最新版本
``` text
示例一：升级所有组件至最新版本
tiup update --all

示例二：升级所有组件至 nightly 版本
tiup update --all --nightly

示例三：升级 TiUP 至最新版本
tiup update --self
```
### tiup clean 清除组件 命令来清理组件实例，并删除工作目录。如果在清理之前实例还在运行，会先 kill 相关进程
- --all 清理所有组件
- tiup clean experiment 清理指定组件

### tiup uninstall 命令来卸载某个组件的所有版本或者特定版本，也支持卸载所有组件
``` bash 
tiup uninstall [component][:version] [flags]
```
- --all：卸载所有的组件或版本
- --self：卸载 TiUP 自身
``` text
示例一：卸载 v5.1.4 版本的 TiDB
tiup uninstall tidb:v5.1.4

示例二：卸载所有版本的 TiKV
tiup uninstall tikv --all

示例三：卸载所有已经安装的组件
tiup uninstall --all
```
### tiup list 查看tiup的各类组件 
``` text
[root@k8s-template ~]# tiup list
Available components:
Name            Owner      Description
----            -----      -----------
PCC             community  A tool used to capture plan changes among different versions of TiDB
bench           pingcap    Benchmark database with different workloads
br              pingcap    TiDB/TiKV cluster backup restore tool
cdc             pingcap    CDC is a change data capture tool for TiDB
chaosd          community  An easy-to-use Chaos Engineering tool used to inject failures to a physical node
client          pingcap    Client to connect playground
cluster         pingcap    Deploy a TiDB cluster for production
ctl             pingcap    TiDB controller suite
dm              pingcap    Data Migration Platform manager
dmctl           pingcap    dmctl component of Data Migration Platform
errdoc          pingcap    Document about TiDB errors
pd-recover      pingcap    PD Recover is a disaster recovery tool of PD, used to recover the PD cluster which cannot start or provide services normally
playground      pingcap    Bootstrap a local TiDB cluster for fun
tidb            pingcap    TiDB is an open source distributed HTAP database compatible with the MySQL protocol
tidb-lightning  pingcap    TiDB Lightning is a tool used for fast full import of large amounts of data into a TiDB cluster
tikv-br         pingcap    TiKV cluster backup restore tool
tiup            pingcap    TiUP is a command-line component management tool that can help to download and install TiDB platform components to the local system
```
- tiup list tidb 查看tidb的可用版本
- --all 查看所有组件，包含隐藏组件
- --installed 显示所有已安装组件
- --verbose 在组件列表中显示已安装的版本列表。默认组件列表不显示当前已安装的版本。

### tiup telemetry tiup遥测功能 
- 执行 TiUP 命令时会将使用情况信息分享给 PingCAP，默认是打开的 
- status 命令 tiup telemetry status 查看当前的遥测设置，输出以下信息：
``` bash 
tiup telemetry status
status: enable  
uuid: 7528b4c4-46c4-4be0-998d-30614aa18409

status: 当前是否开启遥测 (enable|disable)
uuid: 随机生成的遥测标示符
```

- reset 命令 tiup telemetry reset 重置当前的遥测标示符，以一个新的随机标识符代替之。
- enable 命令 tiup telemetry enable 启用遥测。
- disable 命令 tiup telemetry disable 停用遥测。
