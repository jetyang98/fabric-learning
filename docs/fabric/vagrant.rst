****************
vagrant 文件夹
****************

文件夹包含了 Vagrant 工具的配置文件 Vagrantfile 和一些需要的 Bash 脚本文件。

什么是 Vagrant
===============

Vagrant 是一个开源的工具，用于构建和管理虚拟化开发环境。它提供了一个简单而一致的工作流程来创建和配置虚拟机，可以在本地开发环境中快速搭建虚拟机，以便进行软件开发和测试。

Vagrant 可以与多种虚拟化平台集成，包括 VirtualBox、VMware、Hyper-V 等，使开发者能够在这些平台上创建和管理虚拟机。通过 Vagrant，开发者可以定义虚拟机的配置、操作系统和软件包，然后通过简单的命令就能够启动、停止和销毁虚拟机。

Vagrantfile 文件
==================

`文件链接 <https://github.com/hyperledger/fabric/blob/v2.5.5/vagrant/Vagrantfile>`__。

``Vagrant.require_version ">= 1.7.4"``：指定了 Vagrant 的最低版本要求为 1.7.4。这是为了确保使用的 Vagrant 版本符合配置文件中的要求。

``Vagrant.configure('2') do |config|``：配置文件的开始，指定了 Vagrant 配置的版本。在这里，使用的是版本 2 的配置。

``config.vm.box = "bento/ubuntu-20.04"``：指定了虚拟机使用的基础镜像，这里是一个 Ubuntu 20.04 的镜像。Vagrant 会从 Vagrant Cloud 或其他提供者下载这个镜像。

``config.vm.synced_folder "..", "/home/vagrant/fabric"``：设置了共享文件夹，将主机机器上当前目录的上级目录（".."）与虚拟机内的 "/home/vagrant/fabric" 目录进行同步。这样可以方便地在主机和虚拟机之间共享文件。

``config.ssh.forward_agent = true``：启用了 SSH 代理转发，使得虚拟机内的 SSH 代理可以访问主机上的 SSH 密钥。

``config.vm.provider :virtualbox do |vb|``：指定了虚拟机的提供者，这里使用的是 VirtualBox。

``vb.name = "hyperledger"``：设置了虚拟机的名称为 "hyperledger"。

``vb.cpus = 2``：指定了虚拟机的 CPU 核心数为 2。

``vb.memory = 4096``：指定了虚拟机的内存大小为 4096 MB。

下面的 ``config.vm.provision`` 部分是用于配置虚拟机启动时的自动化脚本。每行指定了一个脚本，其中包括了虚拟机的各种配置，如安装必要的软件、设置用户等。这里包括了以下脚本：

- essentials.sh
- docker.sh
- golang.sh
- limits.sh
- softhsm.sh
- user.sh

``# -*- mode: ruby -*- 和 # vi: set ft=ruby``: 是编辑器配置，指定了文件的语法高亮和格式。

这个 Vagrantfile 的目的是创建一个基于 Ubuntu 20.04 的 VirtualBox 虚拟机，配置了共享文件夹、CPU 和内存等参数，并通过脚本进行自动化配置。