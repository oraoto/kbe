+++
title = "Labels"
subtitle = "Kubernetes labels by example"
date = "2017-04-25"
url = "/labels/"
+++

Label（标签）是管理Kubernetes对象的机制。一个Label就是一个键值对，其中会有一些长度和值的[限制](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set), 但是它们没有预定义的含义。所以你可以自由选择合适的Label表达你需要的意思，例如可以用Label表达"这个Pod跑在生成环境"或者“这个Pod属于部门X”。

先创建一个最初只有一个Label（`env=development`）的[Pod](https://github.com/mhausenblas/kbe/blob/master/specs/labels/pod.yaml)：

```bash
$ kubectl create -f https://raw.githubusercontent.com/mhausenblas/kbe/master/specs/labels/pod.yaml

$ kubectl get pods --show-labels
NAME       READY     STATUS    RESTARTS   AGE    LABELS
labelex    1/1       Running   0          10m    env=development
```

上面的`get pods`命令加了`--show-labels`选项，输出结果会加多一列来显示对象的Label。

你可以给一个Pod加上新的Label：

```bash
$ kubectl label pods labelex owner=michael

$ kubectl get pods --show-labels
NAME        READY     STATUS    RESTARTS   AGE    LABELS
labelex     1/1       Running   0          16m    env=development,owner=michael
```

Label可以用来过滤，例如只列出`owner`等于`micheal`的Pod，可以用`--selector`选项：

```bash
$ kubectl get pods --selector owner=michael
NAME      READY     STATUS    RESTARTS   AGE
labelex   1/1       Running   0          27m
```

`--selector`选项可以简写成`-l`，选出`env=development`的Pod：

```bash
$ kubectl get pods -l env=development
NAME      READY     STATUS    RESTARTS   AGE
labelex   1/1       Running   0          27m
```

通常Kubernetes对象还支持集合选择器。启动[另外一个Pod](https://github.com/mhausenblas/kbe/blob/master/specs/labels/anotherpod.yaml)，它有两个Label （`env=production` 和 `owner=michael`）：

```bash
$ kubectl create -f https://raw.githubusercontent.com/mhausenblas/kbe/master/specs/labels/anotherpod.yaml
```

列出Label为`env=development`或`env=production`的Pod：

```bash
$ kubectl get pods -l 'env in (production, development)'
NAME           READY     STATUS    RESTARTS   AGE
labelex        1/1       Running   0          43m
labelexother   1/1       Running   0          3m
```

Label不仅能用在Pod，实际上你可以把Label应用在各种各样的对象上，例如Node（节点）和Service（服务）。
