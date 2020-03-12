# Flr Core Logic Document

## 前言

这是`Flr`的核心逻辑描述文档，用于`Flr`的核心功能的逻辑细节，以确保`Flr`工具系列版本（[CLI版本](https://github.com/Fly-Mix/flr-cli)、[VSCode Extension版本](https://github.com/Fly-Mix/flr-vscode-extension)、[Android Studio Plugin版本](https://github.com/Fly-Mix/flr-as-plugin)）在核心功能上能保持良好的一致性。



## 核心逻辑

`Flr`包括4个核心功能：

- init: 初始化项目
- generate: 根据配置扫描资源目录，为合法的资源添加声明以及生成`r.g.dart`
- start monitor: 启动资源变化监控服务，以监控到有符合条件的资源变化时，调用`generate`功能
- stop monitor: 停止资源变化监控服务

下面将会逐一列出这4个核心功能的核心逻辑。

#### Init

#### Generate

#### Monitor


