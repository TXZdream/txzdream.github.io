# 使用minikube进行单机kubernetes安装

2019/07/12 By TXZdrem

## 前言

为什么要学习kubernetes——Kubernetes 是用于自动部署，扩展和管理容器化应用程序的开源系统。

## 准备工作

### 环境准备

- Linux系统(这里使用的国内的deepin系统，其他环境之间的操作大同小异，这里不一一展示)

- KVM/VirtualBox(这里推荐使用VirtualBox，同时issue中问题的提出和解决基本都是针对vbox的，所以十分建议。当然其他的虚拟软件也有支持的，具体列表见[官方文档](https://kubernetes.io/docs/setup/learning-environment/minikube/#specifying-the-vm-driver))

- Docker(这里和上一项满足其中之一即可，最低要求[1.13.1](https://github.com/kubernetes/kubernetes/pull/77051)，具体安装方法请自行了解)

## 具体步骤

### [安装minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

- 检查虚拟化支持(非虚拟机方案)

    ```(shell)
    egrep --color 'vmx|svm' /proc/cpuinfo
    ```

    输入命令后如果结果非空，则说明机器支持虚拟化，若真的不支持，请尝试使用本机Docker来搭建集群

- [安装kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux)

    ```(shell)
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
    ```

    我在宿舍wifi情况下是完全可以完成访问的，如果不可以的话，可以尝试使用snap来进行安装

    ```(shell)
    sudo snap install kubectl --classic
    ```

    完成安装后，输入`kubectl version`验证是否安装成功

- 安装虚拟机软件

    这是个及其基本的操作了，根据上面给出的列表安装支持的虚拟机软件即可

- 安装minikube

    ```(bash)
    curl -Lo minikube https://storage.googleapis.com/minikube/releases/v1.2.0/minikube-linux-amd64 && chmod +x minikube && sudo cp minikube /usr/local/bin/ && rm minikube
    ```

    当然，如果遇到无法访问的问题，也可以使用github仓库的[release](https://github.com/kubernetes/minikube/releases)来进行安装，并将文件添加执行属性后移动到`PATH`环境变量的路径下，完成上述操作后，可以使用`minikube version`来查看是否正确安装

### [使用minikube搭建集群](https://kubernetes.io/docs/setup/learning-environment/minikube/)

- 启动集群

    ```(shell)
    minikube start --vm-driver=<driver_name> # 该命令大概率无法执行成功
    ```

    例如采用的是vmware环境，所以这里的`driver_name`使用的是`vmware`，但是virtualbox本身就是默认使用的，所以如果使用virtualbox进行配置则只需要start即可。我本地采用的是vmware方案，但是在命令执行完成后，我发现并没有成功，报错信息是这样的：

    ```(text)
    Unable to start VM: new host: Driver "vmware" not found. Do you have the plugin binary "docker-machine-driver-vmware" accessible in your PATH?
    ```

    根据提示我去网上查了这个`docker-machine-driver-vmware`，得到了一个github项目，在release页面完成下载后，将二进制文件的名字改成上面提示的那个，添加执行权限后，移动到`PATH`下面，再次运行启动集群的命令就可以了

    由于kubernetes的一些基础镜像国内是无法访问到的，这就会导致上述命令在配置kubernetes环境的时候失败，因此，我们需要传入kubernetes一些参数，来确保可以配置好环境。至于这些参数的相关说明，大家可以`minikube start-h | grep registry`来查看，这里不进行详细介绍。

    ```(bash)
    minikube start --vm-driver=vmware --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers --registry-mirror http://f1361db2.m.daocloud.io
    ```

    当然这里也可以直接用代理，关于代理的相关使用说明，可以看[官方说明](https://kubernetes.io/docs/setup/learning-environment/minikube/#using-minikube-with-an-http-proxy)，这里不再展开

    完成上面的操作后，由于这时候已经不是第一次启动该虚拟机环境中的kubernetes，所以执行完成后，会显示这样的错误信息

    ```(text)
    error execution phase certs/etcd-ca: failure loading etcd/ca certificate: failed to load certificate: the certificate is not valid yet
    ```

    这里我看了看github的[issue](https://github.com/kubernetes/minikube/issues/2749)，说是在0.27版本就修复了，然而我还是出现了，我想是不是我用的驱动是vmware，然后仔细看了一下错误提示，发现这一行

    ```(text)
    [certs] Using certificateDir folder "/var/lib/minikube/certs/"
    ```

    貌似是先会使用已有的证书，所以我利用`minikube ssh`连接进去，删除了`/var/lib/minikube/certs/`文件夹(这里需要sudo权限)，之后重新运行上面的命令，然后一切顺利，完成了kubernetes环境的安装

## 相关问题处理

1. 无法启动dashboard

    vmware下尚未解决，可以尝试删除原集群后重试，由于官方issue给出的都是在virtuablbox下的解决方式，所以建议更换虚拟机程序进行配置，我也是在更换了virtualbox后实现了一次成功

1. 其他一切证书问题

    建议使用`minikube delete`命令删除集群后再重新创建，确保一次成功，之前的系统镜像和docker镜像都已经有本地缓存，因此不再需要重新下载

## 总结

至此，kubernetes环境的安装全部完成，而其他的进一步的操作，请参考[官方文档](https://kubernetes.io/docs/setup/learning-environment/minikube/)进行学习实践，这里不再展开说明
