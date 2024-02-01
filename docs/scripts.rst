*****************
ci/scripts 目录
*****************

.. contents:: 

在 ci 目录下，只存在一个名为 scripts 的子目录，用于存放在 GitHub Actions 运行时需要调用的 Bash 脚本。

create_binary_package.sh 文件
================================

`文件链接 <https://github.com/hyperledger/fabric/blob/v2.5.5/ci/scripts/create_binary_package.sh>`__。这是一个 Bash 脚本，主要用于构建 Hyperledger Fabric 的发布版本。脚本被 release.yml 文件的 :ref:`Build Fabric Binaries <call create_binary_package.sh>` 调用。

.. code-block:: bash

    #!/bin/bash
    # Copyright IBM Corp. All Rights Reserved.
    #
    # SPDX-License-Identifier: Apache-2.0

指定了脚本应该由哪个解释器来执行。在这里，它指定了 Bash 解释器。后两行是版权声明和 SPDX 许可证标识，用于声明脚本的版权归属和使用许可。

.. code-block:: bash

    set -euo pipefail

设置了 Bash 的一些选项：

:-e: 使脚本在任何命令返回非零退出状态时立即退出。
:-u: 在尝试使用未定义的变量时导致脚本退出。
:-o pipefail: 如果管道中的任何命令失败，则整个管道的退出状态将是失败。

.. code-block:: bash

    make "release/${TARGET}"

运行 make 命令。该命令会调用 :doc:`Makefile <makefile>` 文件中的 release/${TARGET} 目标进行构建。

.. code-block:: bash

    mkdir -p "release/${TARGET}/config"

创建目录 release/${TARGET}/config，用于存放配置文件。

.. code-block:: bash

    # cp not move otherwise this breaks your source tree
    cp sampleconfig/*yaml "release/${TARGET}/config"

    cd "release/${TARGET}"

将 sampleconfig 目录下的所有 YAML 文件复制到 release/${TARGET}/config 目录。切换到 release/${TARGET} 目录。

.. code-block:: bash

    if [ "$TARGET" == "windows-amd64" ]; then
        for FILE in bin/*; do mv $FILE $FILE.exe; done
    fi

如果目标平台是 Windows，执行下面的命令块。将 bin 目录下的所有文件名添加 .exe 扩展名，以符合 Windows 的可执行文件命名规范。

.. code-block:: bash

    # Trim the semrev 'v' from the start of the RELEASE attribute
    VERSION=$(echo $RELEASE | sed -e  's/^v\(.*\)/\1/')

使用 sed 命令从 $RELEASE 变量中提取版本号，并将其保存在 $VERSION 变量中。此操作移除版本号前缀中的 'v' 字符。

.. code-block:: bash

    tar -czvf "hyperledger-fabric-${TARGET}-${VERSION}.tar.gz" bin config builders

创建一个压缩的 tar 文件，包含 bin、config 和 builders 目录。压缩文件的名称格式为 hyperledger-fabric-${TARGET}-${VERSION}.tar.gz，其中 ${TARGET} 是目标平台，${VERSION} 是版本号。

总体来说，这个脚本用于将 Hyperledger Fabric 构建输出的相关文件打包成一个发布版本的压缩文件，以便进一步分发和部署。

evaluate_commits.sh 文件
==========================

`文件链接 <https://github.com/hyperledger/fabric/blob/v2.5.5/ci/scripts/evaluate_commits.sh>`__。主要用于在 Azure Pipelines 中设置一些变量。在 v2.5.5 版本中，该脚本并没有被任何命令调用。

该脚本判断 "docs" 目录是否有修改，并设置了一些用于 Azure Pipelines 的变量。

setup_hsm.sh 文件
===================

`文件链接 <https://github.com/hyperledger/fabric/blob/v2.5.5/ci/scripts/setup_hsm.sh>`__。用于在 Ubuntu 系统上安装 SoftHSM2（软件硬件安全模块）并进行初始化配置。脚本被 release.yml 文件的 :ref:`Unit Tests <unit-tests call setup_hsm_.sh>` 和 :ref:`Integration Tests <integration-tests call setup_hsm_.sh>` 调用。 

.. code-block:: bash

    sudo apt-get install -y softhsm2

使用 apt-get 包管理器安装 SoftHSM2 软件包。-y 参数表示在安装过程中不需要用户确认。

.. code-block:: bash

    sudo mkdir -p /var/lib/softhsm/tokens

创建 SoftHSM2 所需的 token 存储目录。

.. code-block:: bash

    sudo softhsm2-util --init-token --slot 0 --label "ForFabric" --so-pin 1234 --pin 98765432

使用 softhsm2-util 初始化 SoftHSM2 插槽，设置标签为 "ForFabric"，安全操作（SO-PIN）为 1234，用户操作（PIN）为 98765432。

.. code-block:: bash

    sudo chmod -R 777 /var/lib/softhsm

赋予 /var/lib/softhsm 目录及其子目录的完全访问权限，确保 SoftHSM2 具有必要的读写权限。

.. code-block:: bash

    mkdir -p ~/.config/softhsm2

创建用户的 SoftHSM2 配置目录。

.. code-block:: bash

    cp /usr/share/softhsm/softhsm2.conf ~/.config/softhsm2

将系统默认的 SoftHSM2 配置文件复制到用户的配置目录中，以确保正确的 SoftHSM2 配置。
