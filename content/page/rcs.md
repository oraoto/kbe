+++
title = "Replication Controllers"
subtitle = "Kubernetes replication controllers by example"
date = "2017-04-25"
url = "/rcs/"
+++

Replication Controller（RC，副本控制器）是长期运行Pod的管理器。
一个RC会启动指定个数的Pod（称作`replicas（副本）`），并且保证在节点故障或者Pod里的容器出错时副本能持续运行。

下面创建一个[RC](https://github.com/mhausenblas/kbe/blob/master/specs/rcs/rc.yaml)，它会管理一个Pod副本：

```bash
$ kubectl create -f https://raw.githubusercontent.com/mhausenblas/kbe/master/specs/rcs/rc.yaml
```

完成之后就可以看到RC和它的Pod：

```bash
$ kubectl get rc
NAME                DESIRED   CURRENT   READY     AGE
rcex                1         1         1         3m

$ kubectl get pods --show-labels
NAME           READY     STATUS    RESTARTS   AGE    LABELS
rcex-qrv8j     1/1       Running   0          4m     app=sise
```

注意两件事:

- 被管理的Pod会有一个随机的名字（`rcex-qrv8j`）
- RC会通过Label（这里的`app=sise`）来跟踪记录它管理的Pod

要进行扩容，也就是增加副本的个数，可以执行：


```bash
$ kubectl scale --replicas=3 rc/rcex

$ kubectl get pods -l app=sise
NAME         READY     STATUS    RESTARTS   AGE
rcex-1rh9r   1/1       Running   0          54s
rcex-lv6xv   1/1       Running   0          54s
rcex-qrv8j   1/1       Running   0          10m

```

最后，用下面的命令删除掉RC和它管理的Pod：

```bash
$ kubectl delete rc rcex
replicationcontroller "rcex" deleted
```

在这之后，支持集合选择器的RC会被称为[ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)（RS, 副本集）。RS在介绍[Deployment](/kbe/deployments/)的时候就会用到了。
