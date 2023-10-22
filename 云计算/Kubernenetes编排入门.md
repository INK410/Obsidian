> https://kubernetes.io/zh-cn/docs/tasks/tools/install-kubectl-linux/
## 部署minikube

首先，确保你的系统上已安装Docker，因为Minikube使用Docker来运行Kubernetes集群中的节点。然后，按照Minikube的官方文档安装Minikube：

```shell
curl -Lo minikube
https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
mv minikube /usr/local/bin/

// github地址:https://github.com/kubernetes/minikube/releases/download/v1.31.2/minikube-linux-amd64
```

### 步骤2：启动Minikube集群
```shell
minikube start

minikube start --force //root用户执行加--force
```
Minikube将自动下载和启动一个虚拟机，然后在其中创建一个Kubernetes集群。这个过程可能需要一些时间。

### 步骤3：验证Minikube集群
你可以运行以下命令来验证它的状态：
```shell
minikube status
看到输出中的状态为"Running"，表明Minikube集群正常运行。
```

### 步骤四：部署kubectl
获取当前最新kubectl版本号
```shell
curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
//下载最新版kubectl 如需下载某个指定的版本，请用指定版本号替换该命令的这一部分： `$(curl -L -s https://dl.k8s.io/release/stable.txt)`

curl -LO https://dl.k8s.io/release/v1.28.2/bin/linux/amd64/kubectl
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256" //下载校验文件
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check  //kubectl: OK 代表正常
```

#### 安装 kubectl

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
# 之后将 ~/.local/bin 附加（或前置）到 $PATH
```


### 步骤5：使用Minikube
现在，你可以使用Minikube来管理和运行Kubernetes集群。例如，你可以使用kubectl命令行工具与
Minikube集群交互，管理Pod、Service、Deployment等
- 使用以下命令检查Kubernetes集群的状态：
```shell
kubectl cluster-info //如果运行失败可以再执行一次 minikube start --force 
```

- 检测版本
```shell
kubectl version --client
```
如下命令来查看版本的详细信息
```shell
kubectl version --client --output=yaml
```

![[Pasted image 20231021173523.png]]

