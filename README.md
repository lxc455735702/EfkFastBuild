<!--
 * @Description: 
 * @Author: lxc
 * @Date: 2020-06-09 09:17:51
 * @LastEditTime: 2020-06-09 16:11:10
 * @LastEditors: lxc
--> 
# kubernetes efk deploy
1. elasticsearch
```
$ kubectl create -f es-statefulset.yaml -n logging
$ kubectl create -f es-service.yaml -n logging

$ kubectl get pods -n logging
NAME                             READY   STATUS    RESTARTS   AGE
elasticsearch-logging-0          1/1     Running   0          3h26m
elasticsearch-logging-1          1/1     Running   0          3h25m

$ kubectl get svc -n logging
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
elasticsearch-logging   ClusterIP   10.108.178.226   <none>        9200/TCP   3h22m

```
2. kibana
```
$ kubectl create -f kibana.yaml -n logging
$ kubectl create -f kibana-service.yaml -n logging
$ kubectl create -f kibana-ingress.yaml -n logging

$ kubectl get pods -n logging
NAME                             READY   STATUS    RESTARTS   AGE
elasticsearch-logging-0          1/1     Running   0          3h30m
elasticsearch-logging-1          1/1     Running   0          3h29m
kibana-logging-cbc788775-zjc28   1/1     Running   0          3h25m

$ kubectl get svc -n logging
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
elasticsearch-logging   ClusterIP   10.108.178.226   <none>        9200/TCP   3h27m
kibana-logging          ClusterIP   10.98.53.208     <none>        5601/TCP   3h26m
```

3. fluentd

为了能够灵活控制哪些节点的日志可以被收集，所以这里还添加了一个 nodSelector 属性 
```
nodeSelector:
  beta.kubernetes.io/fluentd-ds-ready: "true"
```
想采集节点的日志，那么我们就需要给节点打上上面的标签 我是在所有的节点打上标签 
```
$ kubectl label nodes node名 beta.kubernetes.io/fluentd-ds-ready=true

$ kubectl get nodes --show-labels
NAME              STATUS   ROLES    AGE   VERSION   LABELS
v10-104-141-164   Ready    <none>   69d   v1.17.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/fluentd-ds-ready=true,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=v10-104-141-164,kubernetes.io/os=linux
v10-104-61-249    Ready    master   69d   v1.17.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/fluentd-ds-ready=true,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=v10-104-61-249,kubernetes.io/os=linux,node-role.kubernetes.io/master=
v10-104-61-251    Ready    <none>   69d   v1.17.3   app=ingress,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/fluentd-ds-ready=true,beta.kubernetes.io/os=linux,es=log,kubernetes.io/arch=amd64,kubernetes.io/hostname=v10-104-61-251,kubernetes.io/os=linux
v10-104-61-252    Ready    <none>   69d   v1.17.3   app=ingress,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/fluentd-ds-ready=true,beta.kubernetes.io/os=linux,es=log,kubernetes.io/arch=amd64,kubernetes.io/hostname=v10-104-61-252,kubernetes.io/os=linux
v10-104-61-253    Ready    <none>   69d   v1.17.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/fluentd-ds-ready=true,beta.kubernetes.io/os=linux,es=log,kubernetes.io/arch=amd64,kubernetes.io/hostname=v10-104-61-253,kubernetes.io/os=linux
```
这里需要注意的是，docker的根目录：
```
$ docker info
Docker Root Dir: /home/docker/data
```

所以上面要获取 docker 的容器目录需要更改成/home/docker/data/containers，这个地方非常重要，当然如果你没有更改 docker 根目录则使用默认的/var/lib/docker/containers目录即可

```
$ kubectl create -f fluentd-es-configmap.yaml -n logging
$ kubectl create -f fluentd-es-ds.yaml -n logging
$ kubectl get pods -n logging

NAME                             READY   STATUS    RESTARTS   AGE
elasticsearch-logging-0          1/1     Running   0          3h30m
elasticsearch-logging-1          1/1     Running   0          3h29m
fluentd-es-v2.3.2-2z8n2          1/1     Running   0          3h14m
fluentd-es-v2.3.2-p2j9v          1/1     Running   0          3h14m
fluentd-es-v2.3.2-vqbnz          1/1     Running   0          3h14m
fluentd-es-v2.3.2-xglhj          1/1     Running   0          3h14m
kibana-logging-cbc788775-zjc28   1/1     Running   0          3h25m
```
修改pc hosts 添加 
10.104.61.251 kibana.wyyy.com

访问 http://kibana.wyyy.com/ 即可打开kibana

博客地址：https://blog.csdn.net/github_38787615/article/details/106639641

