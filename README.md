# 说明
容器云部署脚本中包括了kubernetes底层组件、harbor、elfk、metrickbeat、cs容器云组件、gitlab等一系列服务的安装。采用ansbile脚本实现自动安装，运维人员需要对ansible工具有一定简单了解。主要服务基本实现全容器化、k8s化部署，可以通过kubernetes dashboard监控到所有容器服务；部署脚本基于centos 7，要求内核版本为4以上；既提供一键快速安装方式，也提供分步执行安装方式。
# 组件版本
| 名称                    | 版本号       | 备注     |
|-------------------------|--------------|----------|
| Kernel                  | 4以上        |          |
| kube-apiserver          | 1.10.2       |          |
| kube-controller-manager | 1.10.2       |          |
| kube-scheduler          | 1.10.2       |          |
| kube-proxy              | 1.10.2       |          |
| kubelet                 | 1.10.2       |          |
| etcd                    | 3.1.12       |          |
| calico                  | 2.6.7        |          |
| docker                  | 17.06.2-ce   |          |
| kubedns                 | 1.14.8       |          |
| heapster                | 1.5.0        |          |
| kubernets-dashboard     | 1.8.3        |          |
| traefik                 | 1.5.0        |          |
| influxdb                | 1.3.3        |          |
| grafana                 | 4.4.3        |          |
| pause                   | 3.1          |          |
| elasticsearch           | 6.2.4        |          |
| filebeat                | 6.2.4        |          |
| logstash-x-pack         | 6.2.4        |          |
| kibana-x-pack           | 6.2.4        |          |
| metricbeat              | 6.2.3        |          |
| keepalived              | 1.3.9-r2     |          |
| Haproxy                 | 1.8.9-alpine |          |
| Harbor相关              | v1.2.0       |          |
| gluster                 | 3.13.2       |          |
| heketi                  | 6.0          |          |
| metrics-server          | v0.2.0       |          |
| node-problem-detector   | v0.4.1       |          |
| kube-state-metrics      | v1.4.0       |          |
| mysql                   | 5.7.23       |          |
| cs-uimp-cloud-frontend  | 1.6.0        |          |
| cs-uimp-cloud-ccapi     | 1.6.0        |          |
| monitor-service         | 1.6.0        | 告警服务 |
| cs-uimp-cloud-push      | 1.6.0        | 推送服务 |
| cs-uimp-cloud-auth2     | 1.6.0        | 认证服务 |
| cs-inspect              | 1.6.0        | 巡检服务 |
| redis                   | 4.0.2        |          |
| gitlab                  | 8.16.3       |          |
| postgresql              | 9.5-3        |          |
| Openvpn                 | 3.1          |          |

# 快速安装
1. 下载部署文件到部署节点的/etc/ansible目录下
2. 安装ansible  
`rpm -i bin/ansible/*.rpm`
3. 配置集群安装信息  
`cp -f example/hosts.m-masters.example hosts`  
**根据实际情况修改`hosts`文件**
4. 配置ssh免密码  
`sh tools/ssh-key-copy.sh root ${passwd} #请输入实际的root用户密码` 
5. 执行一键安装  
`ansible-playbook setup.yml`

# 分步安装
`playbooks`目录提供分步安装的相关playbook，主要分为两大块`kubernetes`和`gpaas`
## `kubernetes` 部署
1. `ansible-playbook playbooks/kubernetes/01.prepare.yml` 服务器通用配置，生成并分发集群所需相关证书
2. `ansible-playbook playbooks/kubernetes/02.docker.yml` 在所有主机上安装并启动docker服务
3. `ansible-playbook playbooks/kubernetes/03.harbor.yml` 部署harbor节点，安装并启动harbor服务（可选）
4. `ansible-playbook playbooks/kubernetes/04.lb.yml` 准备lb节点所需的相关安装文件，包括keepalived和haproxy
5. `ansible-playbook playbooks/kubernetes/05.kube-master.yml` 准备master节点所需的相关安装文件
6. `ansible-playbook playbooks/kubernetes/06.kube-node.yml` 在主机上安装并启动kubelet服务，先启动lb,再启动master，最后启动kube-node
7. `ansible-playbook playbooks/kubernetes/07.calico.yml` 在主机上准备calico服务所需要的相关安装文件（与flannel可选）
8. `ansible-playbook playbooks/kubernetes/07.flannel.yml` 在主机上准备flannel服务所需要的相关安装文件（与calico可选） （暂时不可用）
9. `ansible-playbook playbooks/kubernetes/09.storage-nfs.yml` 安装nfs服务（与gfs可选）（暂时不可用）
10. `ansible-playbook playbooks/kubernetes/10.storage-gluster.yml` 准备安装gfs服务
11. `ansible-playbook playbooks/kubernetes/20.addnode.yml` 新增节点
12. `ansible-playbook playbooks/kubernetes/30.addons.yml` kubernetes所有插件服务的部署，包括kube-proxy、kubedns、calico、glusterfs等等
13. `ansible-playbook playbooks/kubernetes/90.setup.yml` 一键安装kubernetes,即顺序执行以上所有步骤（除了20.addnode）
14. `ansible-playbook playbooks/kubernetes/99.clean.yml` 一键清理kubernetes集群（慎用）

## `gpass` 部署
目前分为`cs-cloud`、`elk`和`monitor`三部分

### `cs-cloud` 部署
1. `ansible-playbook playbooks/gpaas/cs-cloud/90.setup.yml` 一键安装容器云平台，包括dashboard、ccapi和auth  

### `elk` 部署
1. `ansible-playbook playbooks/gpaas/elk/01.es.yml` es部署
2. `ansible-playbook playbooks/gpaas/elk/02.logstash.yml` logstash部署
3. `ansible-playbook playbooks/gpaas/elk/03.filebeat.yml` filebeat部署
4. `ansible-playbook playbooks/gpaas/elk/04.kibana.yml` kibana部署
5. `ansible-playbook playbooks/gpaas/elk/90.setup.yml` 一键安装elk，即顺序执行以上所有步骤  

### `monitor` 部署
1. `ansible-playbook playbooks/gpaas/elk/01.es.yml` es部署(可选，如果部署了elk则不需要重复部署)
2. `ansible-playbook playbooks/gpaas/monitor/02.metricbeat.yml` metricbeat部署
3. `ansible-playbook playbooks/gpaas/monitor/03.grafana.yml` grafana部署
4. `ansible-playbook playbooks/gpaas/monitor/04.alert.yml` 告警服务部署
5. `ansible-playbook playbooks/gpaas/monitor/05.inspect.yml` 巡检服务部署
6. `ansible-playbook playbooks/gpaas/monitor/06.push.yml` 推送服务部署
7. `ansible-playbook playbooks/gpaas/monitor/90.setup.yml` 一键安装监控平台，即顺序执行以上所有步骤

### 一键部署`gpass`
1. `ansible-playbook playbooks/gpaas/90.setup.yml`
