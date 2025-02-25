---
title: Spec
---
[Overview](config.md) / [Spec](config-spec-en.md)

<br/>

******

# Nocalhost configurations Spec

:::info PRE-REQUIRE

Before starting this section, please make sure you have already known how to configure Nocalhost. If not, please read [Nocalhost Overview](config-overview-en.md) first.

:::

The configuration of Nocalhost can be divided into three parts.

<br/>

******

## [Development Container Configuration](config-dev-container.md)

The first part is the development container configuration, including:

 - The development image
 - The remote directory for file synchronization 
 - The default shell of development container
 - The persistence of development container (Volume)
 - Requests and limits of development container resources
 - Sidecar image customization

:::tip Quickview
```yaml
name: nocalhost-api
serviceType: deployment
containers:
  - name: nocalhost-api
    dev:
      
      image: codingcorp-docker.pkg.coding.net/nocalhost/dev-images/golang:zsh
      workDir: /home/nocalhost-dev
      sidecarImage: nocalhost-docker.pkg.coding.net/nocalhost/public/nocalhost-sidecar:sshversion
      shell: /bin/zsh
      persistentVolumeDirs:
        - path: /the/path/you/want/to/persistent/in/container
          capacity: 10Gi
        - path: /other
          capacity: 10Gi
      storageClass: cbs
      resources:
        limits:
          memory: 4Gi
          cpu: "1"
        requests:
          memory: 2Gi
          cpu: "0.5"
```
:::



<br/>

******

## [Enhance Configuration](config-enhance.md)

The second part is enhance configuration, which is independent of the development container, including:

 - The source code directory for git
 - Whether to automatically turn on the port forwarding after entering `DevMode`
 - File synchronization configuration, including the synchronizing pattern and the ignoring pattern

:::tip Quickview

```yaml
name: nocalhost-api
serviceType: deployment
containers:
  - name: nocalhost-api
    dev:
      
      gitUrl: git@github.com:nocalhost/nocalhost.git
      portForward:
        - 8080:80
        - 3306:3306
      sync:
        type: send
        mode: pattern
        filePattern:
          - .
        ignoreFilePattern:
          - ".git"
          - ".github"
          - ".vscode"
          - "node_modules"
```

:::

<br/>

******

## [Configuration of one-click Running, One-click Debugging and HotReload](config-develop.md)

The third part is the configuration for development process, including:

 - One-click running
 - One-click debugging
 - HotReload

:::tip Quickview

```yaml
name: example
serviceType: deployment
containers:
  - name: you-container
    dev:
      
      command:
        run: [ "./gradlew", "bootRun" ]
        debug: [ "./gradlew", "bootRun", "--debug-jvm" ]
      debug:
        remoteDebugPort: 5005
      hotReload: true
```

:::

