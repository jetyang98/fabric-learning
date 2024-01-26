**********
文件介绍
**********

.. contents:: 

.github 目录
==============

.github ⽬录⽤于存放与 GitHub 配置和功能相关的⽂件。这些⽂件可以⽤于定义仓库的⼯作流程、CI/CD（持续集成/持续部署）、问题模板、贡献指南等。

- **workflows ⽬录**：存放 GitHub Actions ⼯作流程的⽂件。GitHub Actions 允许⾃动化仓库中的⼯作，例如运⾏测试、构建项⽬或部署应⽤程序。⼯作流程⽂件通常存放在 .github/workflows ⽬录下。
- **PULL_REQUEST_TEMPLATE.md ⽂件**：存放⽤于创建拉取请求（Pull Requests）时的模板⽂件，这些模板可以规范化拉取请求的描述和相关信息。

.dockerignore ⽂件
====================

.dockerignore ⽂件⽤于指定在构建 Docker 镜像时需要忽略的⽂件和⽬录，从⽽减⼩最终镜像的⼤⼩，提⾼构建效率，并减少不必要的依赖和⽂件。类似于 .gitignore ⽂件。

.gitattributes ⽂件
=====================

.gitattributes ⽂件是⼀个⽤于指定在 Git 版本控制系统中处理⽂件属性的配置⽂件。它可以⽤来告诉 Git 如何处理特定类型的⽂件，例如⽂本⽂件、⼆进制⽂件等。