部署应用程序的命令式方法是使用该 kubectl create deployment 命令。正如命令本身所建议的那
样，它会创建一个 Deployment 对象，该对象表示群集中部署的应用程序。通过使用命令性命令，可以避免像编写 YAML 或 JSON 清单时那样了解部署对象的结构。

## 步骤一：部署镜像
```shell
kubectl create deployment kubia --image=luksa/kubia:1.0
```