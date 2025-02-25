---
title: Dev Container configuration
---
[Overview](config-en.md) / [Spec](config-spec-en.md) / [Container](config-dev-container.md)

<br/>

******

### 文件同步的远端目录

范例：

```yaml
name: nocalhost-api
serviceType: deployment
containers:
  - name: nocalhost-api
    dev:
      workDir: /home/nocalhost-dev
```

在进入开发模式后，我们要求用户在本地选择并关联一个本地目录，或者用户可以点击指定工作负载右键，选择 `Associate Local DIR` 来主动关联一个本地目录。这个目录的内容会在进入开发模式以后，与容器的 `workDir` 做实时同步。

`workDir` 默认为 `/home/nocalhost-dev`

:::danger workDir 的注意事项

`workDir` 使用了 emptyDir 来在 `container` 中进行共享，所以这个卷所挂载的目录**最初会是空的**。

:::

<br/>

******

### 开发镜像

范例：

```yaml
name: nocalhost-api
serviceType: deployment
containers:
  - name: nocalhost-api
    dev:
      image: codingcorp-docker.pkg.coding.net/nocalhost/dev-images/golang:zsh
```

开发镜像是开发模式工作的基石，可以简单认为是一个远程的 linux，本地同步上去的文件如果想要正确的编译、运行等，必须配合合适的开发镜像。Nocalhost 官方提供了多个开发镜像，如果你没有配置这个字段就进入开发模式，我们会要求你选择或填入一个开发镜像以继续。



官方的开发镜像没有什么特殊的魔改，就是一个普通的镜像。除了各个语言的基本环境，如 Java 的 JRE、Maven 等，还内置了一些如 git、openssh-client、zsh、bash、net-tools、tmux 等基本软件。如果官方的开发镜像不合适，你可以随意的定制自己的开发镜像，我们的 DockerFile 放置在 [dev-container](https://github.com/nocalhost/dev-container)。

:::tip 自制开发镜像

如果你定制了自己的开发镜像，请将它放置在一个你所使用的 K8s 集群可拉取到的仓库。

:::

<br/>

******

### 在远端容器使用什么 Shell

范例：

```yaml
name: nocalhost-api
serviceType: deployment
containers:
  - name: nocalhost-api
    dev:
      shell: /bin/zsh
```

可不进行配置，默认值是 `zsh || bash || sh`。使用便捷易用的 Shell 往往能事半功倍，如带有自动补全、历史记录补全的 zsh。



当然，具体支持什么 Shell 依托于 **开发镜像** 本身，如果你的 **开发镜像** 不支持 zsh，这里即使配置了 zsh 也没有作用，它会依次寻找 zsh、bash 以及 sh，直到找到可用的 Shell。


<br/>


******

### 开发容器持久化

范例：

```yaml
name: nocalhost-api
serviceType: deployment
containers:
  - name: nocalhost-api
    dev:
      persistentVolumeDirs:
        - path: /the/path/you/want/to/persistent/in/container
          capacity: 10Gi
        - path: /other
          capacity: 10Gi
      storageClass: cbs
```

我们知道，K8s 容器中，如果没有对目录进行持久化，在容器停止或重启后，原先产生的数据将会丢失，例如已经同步上去的文件、编译产物、构建产物等等。在开发容器中启用持久化，可以大大减少上述这些时间的损耗。



持久化包括两部分配置：

- 一个是哪些目录需要被持久化，可不进行配置，默认值为空，path 为开发镜像中需要持久化的目录，capacity 为此目录持久化分配的空间，persistentVolumeDirs 是一个数组，可配置多个 path/capacity。
- 另一个是 storageClass。

持久化需要借助 storageclass 的能力（`kubectl get storageclass `），如果没有配置 storageClass，Nocalhost 将会使用集群中默认的 storageclass 来进行 PVC 的创建。反之，则使用指定的 storageClass 来进行创建。

:::info 注意

capacity 需要符合 K8s 对资源限定的约定。

:::

<br/>

******

### 开发容器的资源的申请与限制

范例：

```yaml
name: nocalhost-api
serviceType: deployment
containers:
  - name: nocalhost-api
    dev:
      resources:
        limits:
          memory: 4Gi
          cpu: "1"
        requests:
          memory: 2Gi
          cpu: "0.5"
```

Nocalhost 对开发容器的资源设置继承自原容器，如果原容器没有配置，namespace 中也没有设置对应的 `limitranges` （`kubectl get limitranges`），那么开发容器将开发容器的资源限制为空。



通常，在进入开发模式后，资源的用量会超过原镜像。如果原容器配置了资源相关的设置，往往会导致开发模式无法正常运转，如内存不够导致 OOM 等。那么此时就需要通过配置 `resources` 来提高开发镜像所用资源。

:::info 注意

memory 和 cpu 需要符合 K8s 对资源限定的约定。

:::

<br/>

******

### Sidecar 镜像定制

范例：

```yaml
name: nocalhost-api
serviceType: deployment
containers:
  - name: nocalhost-api
    dev:
      sidecarImage: nocalhost-docker.pkg.coding.net/nocalhost/public/nocalhost-sidecar:sshversion
```

`sidecarImage` 是进入开发模式所必须的镜像，负责代码同步、debug 连接管理等，默认为 `nocalhost-docker.pkg.coding.net/nocalhost/public/nocalhost-sidecar:sshversion`，且不需要手动进行配置。

如果你的集群由于特殊的网络环境无法获取该镜像，可以将当前这个镜像拉取下来，推送到你的集群可以正常访问的镜像仓库，并将其配置为新的地址。

### Patches

范例：

```yaml
name: nocalhost-api
serviceType: deployment
containers:
  - name: nocalhost-api
    dev:
      patches:
        - patch: '{"spec":{"template":{"spec":{"containers":[{"name":"nocalhost-dev","resources":{"limits":{"cpu":"2"}}}]}}}}'
        - patch: '{"spec":{"template":{"spec":{"containers":[{"name":"nocalhost-sidecar","resources":{"limits":{"cpu":"2"}}}]}}}}'
        - patch: '[{"op": "add","path":"/metadata/annotations/nocalhost-patch","value":"hello-world"}]'
          type: json
```

`patches` 配置项提供了类似 `kubectl patch` 的能力，用户可以使用 `patches` 灵活地对进入 Nocalhost DevMode 的工作负载的 Spec 进行适当的修改。 
其中：<br></br>
** &nbsp • ** **type**: 为 patch 的类型，可选的值有： `json`, `merge`, `strategic`, 默认值为 `strategic`<br></br>
** &nbsp • ** **patch**: 为 patch 的内容<br></br>
可以简单地理解为，`type` 配置项等同于 `kubectl patch` 命令的 `--type` 参数，`patch` 配置项等同于 `kubectl patch` 命令的 `--patch` 参数。关于 `kuebctl patch` 命令的具体用法，可以参考 [使用 kubectl patch 更新 K8s API 对象](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/update-api-object-kubectl-patch/)

:::caution
Nocalhost 不会对 patch 的内容做任何的校验和限制，有可能会因为 patch 的内容不当导致 Nocalhost 进入 DevMode 失败，所以请确保 patch 的正确性。
:::