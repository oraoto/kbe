+++
title = "Pods"
subtitle = "Kubernetes pods by example"
date = "2017-08-22"
url = "/pods/"
+++

Pod是一组容器的集合，这些容器会共享同一个网络和挂载命名空间。Pod是Kubernetes部署的基本单位，
一个Pod里的容器会被调度到同一个节点上。

要通过容器[镜像](https://hub.docker.com/r/mhausenblas/simpleservice/)`mhausenblas/simpleservice:0.5.0`来启动一个Pod，并且暴露出一个`9876`端口的HTTP API，可以执行：

```bash
$ kubectl run sise --image=mhausenblas/simpleservice:0.5.0 --port=9876
```

我们可以看到Pod已经在运行了：

```bash
$ kubectl get pods
NAME                      READY     STATUS    RESTARTS   AGE
sise-3210265840-k705b     1/1       Running   0          1m

$ kubectl describe pod sise-3210265840-k705b | grep IP:
IP:                     172.17.0.3
```

从上面的`kubectl describe`命令可以看到这个Pod的IP是 `172.17.0.3`，在集群内（例如，可以通过`minishift ssh`进入）可以通过这个IP访问到这个Pod：

```bash
[cluster] $ curl 172.17.0.3:9876/info
{"host": "172.17.0.3:9876", "version": "0.5.0", "from": "172.17.0.1"}
```

注意了，`kubectl run`会创建一个[部署](/deployments/)，要删除刚才的Pod必须执行
 `kubectl delete deployment sise`。

另外你可以通过配置文件来创建Pod，我们这个[Pod](https://github.com/mhausenblas/kbe/blob/master/specs/pods/pod.yaml)
会跑起一个`simpleservice`容器外加一个通用的`CentOS`容器：

```bash
$ kubectl create -f https://raw.githubusercontent.com/mhausenblas/kbe/master/specs/pods/pod.yaml

$ kubectl get pods
NAME                      READY     STATUS    RESTARTS   AGE
twocontainers             2/2       Running   0          7s
```

我们通过exec进到`CentOS`，然后去访问localhost的`simpleservice`：

```bash
$ kubectl exec twocontainers -c shell -i -t -- bash
[root@twocontainers /]# curl localhost:9876/info
{"host": "localhost:9876", "version": "0.5.0", "from": "127.0.0.1"}
```

声明`resources`字段可以控制一个Pod能使用的CPU和内存量（[这里](https://github.com/mhausenblas/kbe/blob/master/specs/pods/constraint-pod.yaml)限制成64MB内存和0.5个CPU）：

```bash
$ kubectl create -f https://raw.githubusercontent.com/mhausenblas/kbe/master/specs/pods/constraint-pod.yaml

$ kubectl describe pod constraintpod
...
Containers:
  sise:
    ...
    Limits:
      cpu:      500m
      memory:   64Mi
    Requests:
      cpu:      500m
      memory:   64Mi
...
```

Kubernetes的文档有更多关于[资源限制](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-ram-container/)的[资料](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)。

总结一下，在Kubernetes(同时)启动一个或多个容器是很简单的，但是像上面那样直接启动有个很大的限制：
你必须自己处理故障来保持它们正常运行。 一个更好的方法是用[副本控制器](/rcs/)管理Pod，
而更上一层楼的[部署](/deployments)可以给你更多的控制。
