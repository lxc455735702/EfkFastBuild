# kubernetes efk deploy
1. elasticsearch
```
$ create -f es-statefulset.yaml -n logging
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
