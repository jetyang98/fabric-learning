****************
workflows 目录
****************

.. contents:: 

release.yml 文件
===================

`文件链接 <https://github.com/hyperledger/fabric/blob/v2.5.5/.github/workflows/release.yml>`__。这是一个 GitHub Actions 工作流程的 YAML 配置文件，用于自动化 Hyperledger Fabric 项目的发布流程。

.. code-block:: yaml

  # Copyright the Hyperledger Fabric contributors. All rights reserved.
  #
  # SPDX-License-Identifier: Apache-2.0

这部分是文件的版权声明和 SPDX-License-Identifier，表明了文件的版权归属和许可协议。

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

.. _call create_binary_package.sh:

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

slash-commands.yml 文件
=========================

`文件链接 <https://github.com/hyperledger/fabric/blob/v2.5.5/.github/workflows/slash-commands.yml>`__。该文件用于监听 GitHub 仓库的评论事件（issue_comment），并在满足一定条件时执行一系列操作。

.. code-block:: yaml

  name: Slash Commands

定义了这个 GitHub Actions 工作流的名称为 "Slash Commands"。

.. code-block:: yaml

  on:
    issue_comment:
      types:
        - created
        - edited

指定了工作流程的触发条件。即，当有 issue_comment 事件发生时，且事件类型为 created 或 edited 时触发这个工作流。这表明当有人创建或编辑评论时，这个工作流将被触发。

.. code-block:: yaml

  jobs:
    notify:

定义了一个名为 notify 的工作，用于通知并处理不符合要求的问题。这个工作包含一系列步骤（steps），在满足一定条件时执行这些步骤。

.. code-block:: yaml

  name: Invalid Issue Usage
  if: contains(github.event.comment.body, '/invalid') && github.event.issue.state == 'open'

为这个工作设置了一个名为 "Invalid Issue Usage" 的名称，以及一个条件 if。条件表达式使用 GitHub Actions 内置的函数 contains 来检查评论中是否包含 '/invalid'，并且确保问题（issue）的状态为 'open'。只有在这两个条件同时满足时，这个工作才会执行。

.. code-block:: yaml

  runs-on: ubuntu-latest

指定了运行这个工作的操作系统为最新的 Ubuntu 版本。

.. code-block:: yaml

  steps:
    - name: Comment on Issue
      uses: lindluni/issue-manager@v1.0.0
      with:
        action: comment
        message: |
          Thank you for opening this issue.

          GitHub Issues is a tool for tracking bugs, feature requests, and work in general that relates directly to the Fabric codebase. It is not for general help requests. Please use one of the following forums to request help for your issue:

          - Discord: https://discord.com/servers/hyperledger-foundation-905194001349627914
          - Fabric Mailing List: fabric@lists.hyperledger.org

定义了第一个步骤，使用了 lindluni/issue-manager 动作，版本号为 v1.0.0。这个步骤的目的是在相关问题上添加评论，提醒问题提交者正确使用 GitHub Issues。评论内容包括一些信息，说明 GitHub Issues 用于跟踪与 Fabric 代码库直接相关的错误、功能请求等，而不是用于一般性的帮助请求。同时，提供了 Discord 和 Fabric Mailing List 两个论坛用于用户寻求帮助。

.. code-block:: yaml

  - name: Close Issue
    uses: lindluni/issue-manager@v1.0.0
    with:
      action: close

定义了第二个步骤，同样使用了 lindluni/issue-manager 动作。这个步骤的目的是关闭问题。这是一个处理不符合要求的问题的操作，通过添加评论提醒用户并关闭问题，确保 GitHub Issues 被用于其设计目的。

这个工作流程总体上用于检测并处理 GitHub 仓库中的 issue 评论，根据评论内容和问题状态执行相应的操作。

verify-build.yml 文件
=======================

`文件链接 <https://github.com/hyperledger/fabric/blob/v2.5.5/.github/workflows/verify-build.yml>`__。该文件用于执行 Hyperledger Fabric 项目的验证构建，包括基本检查、单元测试和集成测试。

.. code-block:: yaml

  name: Verify Build

定义了这个 GitHub Actions 工作流的名称为 "Verify Build"。

.. code-block:: yaml

  on:
    push:
      branches: ["**"]
    pull_request:
      branches: ["**"]
    workflow_dispatch:

指定了工作流程的触发条件。即，当有推送（push）到任何分支或拉取请求（pull_request）时触发，以及可以手动触发（workflow_dispatch）。

.. code-block:: yaml

  env:
    GOPATH: /opt/go
    PATH: /opt/go/bin:/bin:/usr/bin:/sbin:/usr/sbin:/usr/local/bin:/usr/local/sbin
    GO_VER: 1.21.3

设置了环境变量，包括 GOPATH、PATH、和 GO_VER。这些变量将在后续步骤中使用，用于指定 Go 语言的路径和版本。

.. code-block:: yaml

  jobs:
    basic-checks:
      # ...
    unit-tests:
      # ...
    integration-tests:
      # ...

定义了三个工作（jobs）：

- basic-checks：用于进行基本检查。
- unit-tests：用于执行单元测试。
- integration-tests：用于执行集成测试。

.. code-block:: yaml

  basic-checks:
    name: Basic Checks
    runs-on: ${{ github.repository == 'hyperledger/fabric' && 'fabric-ubuntu-20.04' || 'ubuntu-20.04' }}
    steps:
      - uses: actions/setup-go@v3
        name: Install Go
        with:
          go-version: ${{ env.GO_VER }}
      - uses: actions/checkout@v3
        name: Checkout Fabric Code
      - run: make basic-checks
        name: Run Basic Checks

定义了 basic-checks 工作的步骤，包括安装 Go、检出 Fabric 代码和运行基本检查。

.. _unit-tests call setup_hsm_.sh:

.. code-block:: yaml

  unit-tests:
    name: Unit Tests
    needs: basic-checks
    runs-on: ${{ github.repository == 'hyperledger/fabric' && 'fabric-ubuntu-20.04' || 'ubuntu-20.04' }}
    steps:
      - uses: actions/setup-go@v3
        name: Install Go
        with:
          go-version: ${{ env.GO_VER }}
      - uses: actions/checkout@v3
        name: Checkout Fabric Code
      - run: ci/scripts/setup_hsm.sh
        name: Install SoftHSM
      - run: make unit-test
        name: Run Unit Tests

定义了 unit-tests 工作的步骤，包括安装 Go、检出 Fabric 代码、安装 SoftHSM（软件硬件安全模块），然后运行单元测试。

.. _integration-tests call setup_hsm_.sh:

.. code-block:: yaml

  integration-tests:
    name: Integration Tests
    needs: basic-checks
    strategy:
      fail-fast: false
      matrix:
        INTEGRATION_TEST_SUITE: ["raft","pvtdata","ledger","lifecycle","e2e","discovery gossip devmode pluggable","gateway idemix pkcs11 configtx configtxlator","sbe nwo msp"]
    runs-on: ${{ github.repository == 'hyperledger/fabric' && 'fabric-ubuntu-20.04' || 'ubuntu-20.04' }}
    steps:
      - uses: actions/setup-go@v3
        name: Install Go
        with:
          go-version: ${{ env.GO_VER }}
      - uses: actions/checkout@v3
        name: Checkout Fabric Code
      - run: ci/scripts/setup_hsm.sh
        name: Install SoftHSM
      - run: make integration-test INTEGRATION_TEST_SUITE="${{matrix.INTEGRATION_TEST_SUITE}}"
        name: Run Integration Tests

定义了 integration-tests 工作的步骤，包括安装 Go、检出 Fabric 代码、安装 SoftHSM，然后运行多个集成测试套件，这些套件通过矩阵策略逐个运行。

这个工作流程的目的是在每次推送或拉取请求时验证 Hyperledger Fabric 项目的构建，并进行基本检查、单元测试和集成测试。
