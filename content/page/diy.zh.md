+++
title = "DIY"
subtitle = "Try out for yourself"
date = "2017-08-22"
url = "/diy/"
+++

如果你想动手试试这些例子，可以按下面的步骤配置环境：

1. 安装 [Minishift](https://docs.openshift.org/latest/minishift/getting-started/installing.html)
1. 执行 `minishift start`
1. 执行 `eval $(minishift oc-env)`
1. 执行 `oc login -u system:admin`
1. 创建到`oc`的符号连接 `ln -s oc kubectl`

所有的例子都使用 [oc v3.6.0](https://github.com/openshift/origin/releases/tag/v3.6.0) 和 [Minishift v1.4.1](https://github.com/minishift/minishift/releases/tag/v1.4.1)（即 Kubernetes 1.6） 创建。

```bash
$ minishift version
minishift v1.4.1+0f658ea

$ kubectl version
Client Version: version.Info{Major:"1", Minor:"6", GitVersion:"v1.6.1+5115d708d7", GitCommit:"fff65cf", GitTreeState:"clean", BuildDate:"2017-07-30T21:47:33Z", GoVersion:"go1.7.6", Compiler:"gc", Platform:"windows/amd64"}
Server Version: version.Info{Major:"1", Minor:"6", GitVersion:"v1.6.1+5115d708d7", GitCommit:"fff65cf", GitTreeState:"clean", BuildDate:"2017-08-01T06:24:02Z", GoVersion:"go1.7.6", Compiler:"gc", Platform:"linux/amd64"}
```
