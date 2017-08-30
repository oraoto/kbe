+++
title = "Deployments"
subtitle = "Kubernetes deployments by example"
date = "2017-04-27"
url = "/deployments/"
+++

Deployment是[Pod](/pods/)和[ReplicaSet](/rcs/)的管理器，通过它你可以细粒度地控制一个新的Pod在什么时候、怎样被上线或回滚到前一个状态。

让我们创建一个叫做`site-deploy`的[Deployment](https://github.com/mhausenblas/kbe/blob/master/specs/deployments/d09.yaml)，它会管理两个Pod副本和一个ReplicaSet：

```bash
$ kubectl create -f https://raw.githubusercontent.com/mhausenblas/kbe/master/specs/deployments/d09.yaml
```

完成之后可以查看Deployment、ReplicaSet、Pod：

```bash
$ kubectl get deploy
NAME          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
sise-deploy   2         2         2            2           10s

$ kubectl get rs
NAME                     DESIRED   CURRENT   READY     AGE
sise-deploy-3513442901   2         2         2         19s

$ kubectl get pods
NAME                           READY     STATUS    RESTARTS   AGE
sise-deploy-3513442901-cndsx   1/1       Running   0          25s
sise-deploy-3513442901-sn74v   1/1       Running   0          25s
```

可以看到ReplicaSet和Pod的名字是从Deployment的名字派生出来的。

这个时候，Pod里的`sise`容器被配置成版本`0.9`，我们可以在集群里验证一下（先用`kubectl describe`拿到其中一个Pod的IP）：

```bash
[cluster] $ curl 172.17.0.3:9876/info
{"host": "172.17.0.3:9876", "version": "0.9", "from": "172.17.0.1"}
```

如果我们在一个新的[Deployment](https://github.com/mhausenblas/kbe/blob/master/specs/deployments/d10.yaml)里把版本升到`1.0`，看看会发生什么事：

```bash
$ kubectl apply -f https://raw.githubusercontent.com/mhausenblas/kbe/master/specs/deployments/d10.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment "sise-deploy" configured
```

你也可以用`kubectl edit deploy/sise-deploy`来手工编辑Deployment来实现同样的效果。

我们可以看到有两个版本为`1.0`的Pod上线了，同时两个版本为`0.9`的Pod被结束了：

```bash
$ kubectl get pods
NAME                           READY     STATUS        RESTARTS   AGE
sise-deploy-2958877261-nfv28   1/1       Running       0          25s
sise-deploy-2958877261-w024b   1/1       Running       0          25s
sise-deploy-3513442901-cndsx   1/1       Terminating   0          16m
sise-deploy-3513442901-sn74v   1/1       Terminating   0          16m
```

同时，Deployment还创建了一个新的ReplicaSet：

```bash
$ kubectl get rs
NAME                     DESIRED   CURRENT   READY     AGE
sise-deploy-2958877261   2         2         2         4s
sise-deploy-3513442901   0         0         0         24m
```

在部署的过程中，可以通过`kubectl rollout status deploy/sise-deploy`来查看部署的进度。

要验证一下`1.0`版本真的上线了，我们可以在集群里执行（同样要先`kubectl describe`拿到一个Pod的IP）：

```bash
[cluster] $ curl 172.17.0.5:9876/info
{"host": "172.17.0.5:9876", "version": "1.0", "from": "172.17.0.1"}
```

全部部署历史可以通过下面的命令获取：

```bash
$ kubectl rollout history deploy/sise-deploy
deployments "sise-deploy"
REVISION        CHANGE-CAUSE
1               <none>
2               <none>
```

如果部署过程中出现问题，Kubenetes会自动回滚到前一个版本，当然你可以指定要回滚到的版本（例如这里会回滚到版本1，也就是最初的版本）：

```bash
$ kubectl rollout undo deploy/sise-deploy --to-revision=1
deployment "sise-deploy" rolled back

$ kubectl rollout history deploy/sise-deploy
deployments "sise-deploy"
REVISION        CHANGE-CAUSE
2               <none>
3               <none>

$ kubectl get pods
NAME                           READY     STATUS    RESTARTS   AGE
sise-deploy-3513442901-ng8fz   1/1       Running   0          1m
sise-deploy-3513442901-s8q4s   1/1       Running   0          1m
```

这时，我们就回到了最初的版本，两个Pod都是`0.9`。

最后，我们把Deployment和它管理的ReplicaSet、Pod都清理掉：

```bash
$ kubectl delete deploy sise-deploy
deployment "sise-deploy" deleted
```

在[文档](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)里可以找到更多Deployment的选项。

