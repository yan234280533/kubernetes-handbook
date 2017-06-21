# kubectl

kubectl是Kubernetes的命令行工具（CLI），是Kubernetes用户和管理员必备的管理工具。

kubectl提供了大量的子命令，方便管理Kubernetes集群中的各种功能。这里不再罗列各种子命令的格式，而是介绍下如何查询命令的帮助

- `kubectl -h`查看子命令列表
- `kubectl options`查看全局选项
- `kubectl <command> --help`查看子命令的帮助
- `kubectl [command] [PARAMS] -o=<format>`设置输出格式（如json、yaml、jsonpath等）

## 配置

使用kubectl的第一步是配置Kubernetes集群以及认证方式，包括

- cluster信息：Kubernetes server地址
- 用户信息：用户名、密码或密钥
- Context：cluster、用户信息以及Namespace的组合

示例

```sh
kubectl config set-credentials myself --username=admin --password=secret
kubectl config set-cluster local-server --server=http://localhost:8080
kubectl config set-context default-context --cluster=local-server --user=myself --namespace=default
kubectl config use-context default-context
kubectl config view
```

## 常用命令格式

- 创建：`kubectl run <name> --image=<image>`或者`kubectl create -f manifest.yaml`
- 查询：`kubectl get <resource>`
- 更新
- 删除：`kubectl delete <resource> <name>`或者`kubectl delete -f manifest.yaml`
- 查询Pod IP：`kubectl get pod <pod-name> -o jsonpath='{.status.podIP}'`
- 容器内执行命令：`kubectl exec -ti <pod-name> sh`
- 容器日志：`kubectl logs [-f] <pod-name>`
- 导出服务：`kubectl expose deploy <name> --port=80`

注意，`kubectl run`仅支持Pod、Replication Controller、Deployment、Job和CronJob等几种资源。具体的资源类型是由参数决定的，默认为Deployment：

|创建的资源类型|参数|
|------------|---|
|Pod|`--restart=Never`|
|Replication Controller|`--generator=run/v1`|
|Deployment|`--restart=Always`|
|Job|`--restart=OnFailure`|
|CronJob|`--schedule=<cron>`|

## 命令行自动补全

Linux系统Bash：

```sh
source /usr/share/bash-completion/bash_completion
source <(kubectl completion bash)
```

MacOS zsh

```sh
source <(kubectl completion zsh)
```

## 日志查看
`kubectl logs`用于显示pod运行中，容器内程序输出到标准输出的内容。跟docker的logs命令类似。
```sh
  # Return snapshot logs from pod nginx with only one container
  kubectl logs nginx
  
  # Return snapshot of previous terminated ruby container logs from pod web-1
  kubectl logs -p -c ruby web-1
  
  # Begin streaming the logs of the ruby container in pod web-1
  kubectl logs -f -c ruby web-1
```

## 连接到一个正在运行的容器
`kubectl attach`用于连接到一个正在运行的容器。跟docker的attach命令类似。
```sh
  # Get output from running pod 123456-7890, using the first container by default
  kubectl attach 123456-7890
  
  # Get output from ruby-container from pod 123456-7890
  kubectl attach 123456-7890 -c ruby-container
  
  # Switch to raw terminal mode, sends stdin to 'bash' in ruby-container from pod 123456-7890
  # and sends stdout/stderr from 'bash' back to the client
  kubectl attach 123456-7890 -c ruby-container -i -t

Options:
  -c, --container='': Container name. If omitted, the first container in the pod will be chosen
  -i, --stdin=false: Pass stdin to the container
  -t, --tty=false: Stdin is a TTY
```
## 在容器内部执行命令
`kubectl exec`用于在一个正在运行的容器执行命令。跟docker的exec命令类似。
```sh
  # Get output from running 'date' from pod 123456-7890, using the first container by default
  kubectl exec 123456-7890 date
  
  # Get output from running 'date' in ruby-container from pod 123456-7890
  kubectl exec 123456-7890 -c ruby-container date
  
  # Switch to raw terminal mode, sends stdin to 'bash' in ruby-container from pod 123456-7890
  # and sends stdout/stderr from 'bash' back to the client
  kubectl exec 123456-7890 -c ruby-container -i -t -- bash -il

Options:
  -c, --container='': Container name. If omitted, the first container in the pod will be chosen
  -p, --pod='': Pod name
  -i, --stdin=false: Pass stdin to the container
  -t, --tty=false: Stdin is a TT
```

## 端口转发

`kubectl port-forward`用于将本地端口转发到指定的Pod。

```sh
# Listen on port 8888 locally, forwarding to 5000 in the pod
kubectl port-forward mypod 8888:5000
```

## kubectl proxy

kubectl proxy命令提供了一个Kubernetes API服务的HTTP代理。

```sh
$ kubectl proxy --port=8080
Starting to serve on 127.0.0.1:8080
```

可以通过代理地址`http://localhost:8080/api/`来直接访问Kubernetes API，比如查询Pod列表

```sh
curl http://localhost:8080/api/v1/namespaces/default/pods
```

## kubectl drain

```
kubectl drain NODE [Options]
```

- 它会删除该NODE上由ReplicationController, ReplicaSet, DaemonSet, StatefulSet or Job创建的Pod
- 不删除mirror pods（因为不可通过API删除mirror pods）
- 如果还有其它类型的Pod（比如不通过RC而直接通过kubectl create的Pod）并且没有--force选项，该命令会直接失败
- 如果命令中增加了--force选项，则会强制删除这些不是通过ReplicationController, Job或者DaemonSet创建的Pod

有的时候不需要evict pod，只需要标记Node不可调用，可以用`kubectl cordon`命令。

恢复的话只需要运行`kubectl uncordon NODE`将NODE重新改成可调度状态。

## 附录

kubectl的安装方法

```sh
# OS X
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl

# Linux
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

# Windows
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/windows/amd64/kubectl.exe
```
