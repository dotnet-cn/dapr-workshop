# 作业 2 - 添加 Dapr 服务调用

在这个作业中，您将把 Dapr 添加到组合中。您将使用 Dapr**服务调用**构建块。

## Dapr 服务调用构建块

在微服务应用中，重要的是能够在不知道其他服务所在位置的情况下与其他服务进行通信，特别是当服务在 Kubernetes（或其他一些编排平台）中运行时，服务可以随时移动，或者替换为新版本。这就是 Dapr 服务调用构建块的用武之地。它的工作原理如下：


<img src="img/service-invocation.png" style="zoom: 33%;">

在 Dapr 中，每个服务都以一个唯一的 Id（ *app-id* ）启动，可用于查找到该服务。假设服务 A 想要呼叫服务 B。

1. 服务 A 在其 Dapr sidecar 上调用 Dapr 服务调用 API（使用 HTTP 或 gRPC）并指定服务 B 的唯一 app-id。
2. Dapr 使用其名称解析组件来发现服务 B 的位置，用于找到解决方案正在运行中的托管环境。
3. 服务 A 的 Dapr sidecar将消息转发到服务 B 的 Dapr sidecar。
4. 服务 B 的 Dapr sidecar 将请求转发给服务 B，服务 B 执行相应的业务逻辑。
5. 服务 B 向其 Dapr sidecar 返回服务 A 的响应。
6. 服务 B 的 Dapr  sidecar将响应转发到服务 A 的 Dapr  sidecar。
7. 服务 A 的 Dapr sidecar 将响应转发给服务 A。

对于此动手作业，这是您需要了解的有关此构建块的全部信息。

> 该构建块提供了更多功能，例如安全性和负载平衡。稍后您可以查看 Dapr 文档来了解有关这些附加功能的所有信息。

## 作业目标

要完成此任务，您必须达到以下目标：

- VehicleRegistrationService 和 FineCollectionService 都使用 Dapr sidecar 运行。
- FineCollectionService 使用 Dapr 服务调用构建块来调用`/vehicleinfo/{licensenumber}`终端。

此作业是end-state设置中的**第 1 项：**


<img src="../img/dapr-setup.png" style="zoom: 67%;">

## DIY说明

在 VS Code 中打开源代码文件夹，然后打开[Dapr 服务调用文档](https://docs.dapr.io/developing-applications/building-blocks/service-invocation/)。如果您需要任何提示，您可以查看分步说明。

## 分步说明

要获取实现目标的分步说明，请打开分步说明：

- .NET版本
- Java版本
- Python版本

## 下一个任务

在继续下一个任务之前，请确保停止所有正在运行的进程并关闭 VS Code 中的所有终端窗口。

跳到[作业 3](../Assignment03/README.md) 。
