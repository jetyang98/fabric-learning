***************
Makefile 文件
***************

.. contents:: 

`文件链接 <https://github.com/hyperledger/fabric/blob/v2.5.5/Makefile>`__。Makefile 文件提供了对项目的多种操作命令，例如 clean（清除 build 文件夹）、basic-checks（执行基本检查）、orderer（构建本地 orderer 二进制文件）、unit-test（单元测试） 等。

什么是 Makefile 文件
=====================

Makefile 是一种文本文件，通常用于定义项目的构建规则和依赖关系，以及描述如何生成一个或多个目标文件。它是使用 make 命令执行，make 工具根据 Makefile 中的规则来构建和更新项目的文件。

一个 Makefile 包含一系列规则，每个规则描述了一个或多个目标文件、它们的依赖关系以及如何生成这些目标文件的命令。make 工具会根据这些规则来判断哪些目标需要重新构建，然后执行相应的命令。

一个简单的 Makefile 的结构如下：

.. code-block:: makefile

    target: dependencies
        command

- target：要生成的文件的名称或者伪命令。
- dependencies：生成 target 所需的文件或其他目标。
- command：生成 target 的命令。

Makefile 文档
===============

- Make 命令教程：https://www.ruanyifeng.com/blog/2015/02/make.html。
- 官方文档：https://www.gnu.org/software/make/manual/make.html。

关键代码介绍
=============

.. code-block:: makefile
    :lineno-start: 86

    RELEASE_EXES = orderer $(TOOLS_EXES)
    RELEASE_IMAGES = baseos ccenv orderer peer tools
    RELEASE_PLATFORMS = darwin-amd64 darwin-arm64 linux-amd64 linux-arm64 windows-amd64
    TOOLS_EXES = configtxgen configtxlator cryptogen discover ledgerutil osnadmin peer

.. code-block:: makefile
    :lineno-start: 267

    .PHONY: $(RELEASE_PLATFORMS:%=release/%)
    $(RELEASE_PLATFORMS:%=release/%): GO_LDFLAGS = $(METADATA_VAR:%=-X $(PKGNAME)/common/metadata.%)
    $(RELEASE_PLATFORMS:%=release/%): release/%: $(foreach exe,$(RELEASE_EXES),release/%/bin/$(exe))
    $(RELEASE_PLATFORMS:%=release/%): release/%: ccaasbuilder/%

    # explicit targets for all platform executables
    $(foreach platform, $(RELEASE_PLATFORMS), $(RELEASE_EXES:%=release/$(platform)/bin/%)):
        $(eval platform = $(patsubst release/%/bin,%,$(@D)))
        $(eval GOOS = $(word 1,$(subst -, ,$(platform))))
        $(eval GOARCH = $(word 2,$(subst -, ,$(platform))))
        @echo "Building $@ for $(GOOS)-$(GOARCH)"
        mkdir -p $(@D)
        GOOS=$(GOOS) GOARCH=$(GOARCH) go build -o $@ -tags "$(GO_TAGS)" -ldflags "$(GO_LDFLAGS)" -buildvcs=false $(pkgmap.$(@F))

这部分代码由 create_binary_package.sh 脚本中的 ``make "release/${TARGET}"`` 调用。

267行：声明了一个伪目标。伪目标表示一些操作，而不是实际的文件。在这里，``$(RELEASE_PLATFORMS:%=release/%)`` 是所有平台构建目标的集合。

268行：为了构建每个平台，使用了 LDFLAGS（链接标志），这里通过 ``GO_LDFLAGS`` 变量设置了一些版本和元数据信息，例如 Fabric 版本号、提交哈希等。

269行：对于每个平台目标，有一个依赖关系，这些依赖关系包括该平台的所有二进制文件，这些文件是通过 ``$(RELEASE_EXES)`` 中的目标生成的。

270行：另一个依赖关系。

273行：这是一个嵌套的 foreach 循环，用于为每个平台中的每个二进制文件生成构建目标。

274行：这里使用 eval 函数定义了一个新变量 ``platform``，该变量包含当前构建目标的平台信息。

275行：从 ``platform`` 中提取了操作系统 ``GOOS`` 信息。

276行：提取了体系结构 ``GOARCH`` 信息。

277行：输出正在构建的目标信息。

278行：如果不存在，则创建目标的文件夹。

279行：这是实际的构建命令。通过设置适当的环境变量 ``GOOS`` 和 ``GOARCH``，调用 ``go build`` 构建了二进制文件。
