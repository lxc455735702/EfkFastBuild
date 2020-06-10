<!--
 * @Description: 
 * @Author: lxc
 * @Date: 2020-06-09 09:17:51
 * @LastEditTime: 2020-06-10 18:18:01
 * @LastEditors: lxc
--> 
# kubernetes efk deploy
1. elasticsearch
```
$ kubectl create -f es-statefulset.yaml -n logging
$ kubectl create -f es-svc.yaml -n logging

$ kubectl get pods -n logging
NAME                      READY   STATUS    RESTARTS   AGE
es-0                      1/1     Running   0          6h9m
es-1                      1/1     Running   0          6h9m
es-2                      1/1     Running   0          6h8m

ps node label

$ kubectl get svc -n logging
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
elasticsearch   ClusterIP   None            <none>        9200/TCP,9300/TCP   6h17m


```
2. kibana
```
$ kubectl create -f kibana.yaml -n logging

$ kubectl get pods -n logging
NAME                      READY   STATUS    RESTARTS   AGE
es-0                      1/1     Running   0          6h9m
es-1                      1/1     Running   0          6h9m
es-2                      1/1     Running   0          6h8m
kibana-7fc6d9dcbf-9twkd   1/1     Running   0          5h22m

$ kubectl get svc -n logging
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
elasticsearch   ClusterIP   None            <none>        9200/TCP,9300/TCP   6h9m
kibana          NodePort    10.108.169.43   <none>        5601:32355/TCP      5h14m
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
NAME                      READY   STATUS    RESTARTS   AGE
es-0                      1/1     Running   0          6h14m
es-1                      1/1     Running   0          6h13m
es-2                      1/1     Running   0          6h13m
fluentd-es-v2.3.2-64phl   1/1     Running   0          5h18m
fluentd-es-v2.3.2-fqtpf   1/1     Running   0          5h18m
fluentd-es-v2.3.2-gjk72   1/1     Running   0          5h18m
fluentd-es-v2.3.2-m8bv5   1/1     Running   0          5h18m
fluentd-es-v2.3.2-rb6z6   1/1     Running   0          5h18m
kibana-7fc6d9dcbf-9twkd   1/1     Running   0          5h26m

```

访问 10.104.61.249:32355 即可打开kibana

如果对于搭建 efk 有所帮助，欢迎Star，谢谢。


