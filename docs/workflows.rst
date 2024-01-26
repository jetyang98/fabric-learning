****************
workflows 目录
****************

.. contents:: 

release.yml 文件
===================

这是一个 GitHub Actions 工作流程的 YAML 配置文件，用于自动化 Hyperledger Fabric 项目的发布流程。

.. code-block:: yaml

  name: Release

指定了这个 GitHub Actions 工作流的名称为 "Release"。

.. code-block:: yaml
  
  on:
    push:
      tags: [ v2.* ]

定义了工作流程触发条件，即在推送（push）并且标签（tags）以 "v2." 开头的情况下触发。

.. code-block:: yaml

  env:
    GO_VER: 1.21.3
    UBUNTU_VER: 20.04
    FABRIC_VER: ${{ github.ref_name }}
    DOCKER_REGISTRY: ${{ github.repository_owner == 'hyperledger' && 'docker.io' || 'ghcr.io' }}

设置了环境变量，其中包括 Golang 版本（GO_VER）、Ubuntu 版本（UBUNTU_VER）、Fabric 版本（FABRIC_VER）和 Docker Registry（DOCKER_REGISTRY）。

.. code-block:: yaml

  permissions:
    contents: read

指定了工作流程对内容的读取权限。

.. code-block:: yaml

  jobs:
    build-binaries:
      # ...
    build-and-push-docker-images:
      # ...
    create-release:
      # ...

定义了三个工作（jobs）：

- build-binaries：用于构建 Fabric 二进制文件。
- build-and-push-docker-images：用于构建并推送 Docker 镜像。
- create-release：用于创建 GitHub Release。

每个工作都包含一系列步骤（steps），用于执行具体的任务：

- 工作 build-binaries 包含了步骤来安装 Go、检出 Fabric 代码、编译二进制文件并创建压缩包，最后将生成的压缩包发布为工件。
- 工作 build-and-push-docker-images 包含了步骤来设置 QEMU 和 Docker Buildx，然后通过 Docker Buildx 构建并推送各个组件的 Docker 镜像。
- 工作 create-release 需要前两个工作完成后执行，包含了步骤来检出 Fabric 代码、下载构建产物（artifacts）并使用 ncipollo/release-action 动作创建 GitHub Release。

此工作流程整体上用于自动执行发布流程，从构建二进制文件、构建 Docker 镜像，到创建 GitHub Release，实现了一系列自动化操作。
