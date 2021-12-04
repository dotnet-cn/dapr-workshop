# 作业 1 - 运行应用程序

在本作业中，您将运行应用程序以确保一切工作正常。

## 作业目标

要完成此任务，您必须达到以下目标：

- 所有服务都在运行。
- 日志记录表明所有服务都在正常工作。

提醒一下，这就是服务之间的交互方式：


<img src="../img/services.png" style="zoom: 67%;">

## 步骤 1. 运行 VehicleRegistration 服务

1. 在 VS Code 中打开源代码文件夹。本指南假设使用 VS Code，但您可以随意使用您喜欢的编辑器或 IDE。

    > 在整个过程中，您可以在编辑器或 IDE 窗口的同一实例中执行所有步骤。

2. 打开终端窗口。

    > 您可以使用热键`Ctrl-`` (Windows) 或`Shift-Ctrl-`` (macOS) 执行此操作。

3. 确保当前文件夹是`VehicleRegistrationService` 。

4. `mvn spring-boot:run`启动服务。

> 如果您在此处收到错误消息，请仔细检查您是否已安装工作坊的[所有预装软件！](../README.md#Prerequisites)

现在您可以测试是否可以调用 VehicleRegistrationService。您可以使用浏览器、cURL 或其他一些 HTTP 客户端来执行此操作。但是有一种直接从 VS Code 测试 RESTful API 的便捷方法（使用 VS Code的REST Client扩展 ）：

1. 在编辑器中打开文件`VehicleRegistrationService/test.http`。此文件中的请求模拟获得某个车牌号的车辆和所有者信息。

2. 在文件中点击`Send request`发送到API的请求：

    ![REST客户端](img/rest-client.png)

3. 请求的响应将显示在右侧的单独窗口中。它应该是一个带有 HTTP 状态码`200 OK`的响应，并且正文应该包含一些随机的车辆和所有者信息：

    ```json
    HTTP/1.1 200
    Connection: keep-alive
    Content-Type: application/json
    Date: Wed, 16 Jun 2021 19:39:05 GMT
    Keep-Alive: timeout=60
    Transfer-Encoding: chunked

    {
        "vehicleId": "KZ-49-VX",
        "brand": "Toyota",
        "model": "Rav 4",
        "ownerName": "Angelena Fairbairn",
        "ownerEmail": "angelena.fairbairn@outlook.com"
    }
    ```

4. 检查终端窗口中的日志记录。它应该是这样的：

    ```console
    2021-09-15 13:42:01.030  INFO 14048 --- [           main] d.v.VehicleRegistrationApplication       : Started VehicleRegistrationApplication in 2.031 seconds (JVM running for 2.386)
    2021-09-15 13:42:11.519  INFO 14048 --- [nio-6002-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
    2021-09-15 13:42:11.519  INFO 14048 --- [nio-6002-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
    2021-09-15 13:42:11.520  INFO 14048 --- [nio-6002-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
    2021-09-15 13:42:11.544  INFO 14048 --- [nio-6002-exec-1] dapr.vehicle.VehicleInfoController       : Retrieving vehicle-info for license number KZ-49-VX
    ```

## 步骤 2. 运行 FineCollection 服务

1. 确保 VehicleRegistrationService 服务正在运行（步骤 1 的结果）。

2. 在 VS Code 中打开一个**新的终端窗口。**

    > 您可以使用热键做到这一点，在Windows上使用 `Ctrl-``，在MacOS使用`Shift-Ctrl-``，或点击终端窗口的标题栏中的`+`按钮：<br>![](img/terminal-new-java.png)

3. 确保当前文件夹是`FineCollectionService` 。

4. 使用`mvn spring-boot:run`启动服务。

5. 在 VS Code 中打开文件`FineCollectionService/test.http`。此文件中的请求模拟向 FineCollectionService 发送检测到的超速违规。

6. 单击`Execute request`以向 API 发送请求。

7. 请求的响应将显示在右侧的单独窗口中。它应该是 HTTP 状态码`200 OK`且没有正文的响应。

8. 检查终端窗口中的日志记录。它应该是这样的：

    ```console
    2021-09-15 13:43:58.766  INFO 16412 --- [           main] dapr.fines.FineCollectionApplication     : Started FineCollectionApplication in 2.033 seconds (JVM running for 2.422)
    2021-09-15 13:44:02.464  INFO 16412 --- [nio-6001-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
    2021-09-15 13:44:02.464  INFO 16412 --- [nio-6001-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
    2021-09-15 13:44:02.465  INFO 16412 --- [nio-6001-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
    2021-09-15 13:44:02.613  INFO 16412 --- [nio-6001-exec-1] dapr.fines.violation.ViolationProcessor  : Sent fine notification
             To Genevieve Burnside, registered owner of license number RT-318-K.
             Violation of 15 km/h detected on the A12 road on September, 20 2020 at 08:33:41.
             Fine: EUR 130.00.
    ```

## 步骤 3. 运行 TrafficControl 服务

1. 确保 VehicleRegistrationService 和 FineCollectionService 正在运行（步骤 1 和 2 的结果）。

2. 在 VS Code 中打开一个**新的**终端窗口并确保当前文件夹是`TrafficControlService` 。

3. 使用`mvn spring-boot:run`启动服务。

4. 在 VS Code 中打开`TrafficControlService/test.http`

5. 单击`Execute request`以向 API 发送两个请求。

6. 请求的响应将显示在右侧的单独窗口中。两个请求都应产生 HTTP 状态码`200 OK`且没有正文的响应。

7. 检查终端窗口中的日志记录。它应该是这样的：

    ```console
    2021-09-15 13:45:41.497  INFO 22453 --- [           main] dapr.traffic.TrafficControlApplication   : Started TrafficControlApplication in 1.812 seconds (JVM running for 2.187)
    2021-09-15 13:46:01.675  INFO 22453 --- [nio-6000-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
    2021-09-15 13:46:01.675  INFO 22453 --- [nio-6000-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
    2021-09-15 13:46:01.676  INFO 22453 --- [nio-6000-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 0 ms
    2021-09-15 13:46:01.757  INFO 22453 --- [nio-6000-exec-1] dapr.traffic.TrafficController           : ENTRY detected in lane 1 at 10:38:47 of vehicle with license number XT-346-Y.
    2021-09-15 13:46:03.537  INFO 22453 --- [nio-6000-exec-2] dapr.traffic.TrafficController           :  EXIT detected in lane 1 at 10:38:53 of vehicle with license number XT-346-Y.
    2021-09-15 13:46:05.969  INFO 22453 --- [nio-6000-exec-3] dapr.traffic.TrafficController           :  EXIT detected in lane 1 at 10:38:52 of vehicle with license number XT-346-Y.
    2021-09-15 13:46:05.970  INFO 22453 --- [nio-6000-exec-3] dapr.traffic.TrafficController           : Speeding violation by vehicle XT-346-Y detected: 15 km/h
    ```

8. 还要检查 FineCollectionService 的日志记录。

    > 您可以通过在终端窗口右侧的**选项卡视图中**单击来选择另一个终端窗口：<br>![](img/terminal-tabs-java.png)

    您应该会看到 FineCollectionService 正在处理的超速违规：

    ```console
    2021-09-15 13:46:06.024  INFO 16412 --- [nio-6001-exec-2] dapr.fines.violation.ViolationProcessor  : Sent fine notification
             To Rosa Goeke, registered owner of license number XT-346-Y.
             Violation of 15 km/h detected on the A12 road on September, 10 2020 at 10:38:52.
             Fine: EUR 130.00.
    ```

## 步骤 4. 运行模拟

您已经使用 REST Client直接测试了 API。现在您将运行的模拟是实际模拟汽车在高速公路上行驶的，该模拟将模拟 3 个入口和出口的摄像头（每个车道一个）。

1. 在 VS Code 中打开一个新的终端窗口，并确保当前文件夹是`Simulation` 。

2. 使用`mvn spring-boot:run`启动服务。

3. 在模拟窗口中，您应该看到如下内容：

    ```console
    2021-09-15 13:47:59.599  INFO 22875 --- [           main] dapr.simulation.SimulationApplication    : Started SimulationApplication in 0.98 seconds (JVM running for 1.289)
    2021-09-15 13:47:59.603  INFO 22875 --- [pool-1-thread-2] dapr.simulation.Simulation               : Start camera simulation for lane 1
    2021-09-15 13:47:59.603  INFO 22875 --- [pool-1-thread-1] dapr.simulation.Simulation               : Start camera simulation for lane 0
    2021-09-15 13:47:59.603  INFO 22875 --- [pool-1-thread-3] dapr.simulation.Simulation               : Start camera simulation for lane 2
    2021-09-15 13:47:59.679  INFO 22875 --- [pool-1-thread-2] dapr.simulation.Simulation               : Simulated ENTRY of vehicle with license number 77-ZK-59 in lane 1
    2021-09-15 13:47:59.869  INFO 22875 --- [pool-1-thread-3] dapr.simulation.Simulation               : Simulated ENTRY of vehicle with license number LF-613-D in lane 2
    2021-09-15 13:48:00.852  INFO 22875 --- [pool-1-thread-1] dapr.simulation.Simulation               : Simulated ENTRY of vehicle with license number 12-LZ-KS in lane 0
    2021-09-15 13:48:04.797  INFO 22875 --- [pool-1-thread-2] dapr.simulation.Simulation               : Simulated  EXIT of vehicle with license number 77-ZK-59 in lane 0
    2021-09-15 13:48:04.894  INFO 22875 --- [pool-1-thread-3] dapr.simulation.Simulation               : Simulated  EXIT of vehicle with license number LF-613-D in lane 0
    ```

4. 还要检查所有其他终端窗口中的日志记录。您应该在日志记录中看到检测到的所有进入和退出事件以及任何超速违规行为。

现在我们知道应用程序运行正常。是时候开始将 Dapr 添加到应用程序中了。

## 下一个任务

在继续下一个任务之前，请确保停止所有正在运行的进程并关闭 VS Code 中的所有终端窗口。通过在终端窗口中按`Ctrl-C`来停止服务或模拟。要关闭终端窗口，请输入`exit`命令。

> 您可以通过单击标题栏中的垃圾桶图标来快速关闭终端窗口：![](img/terminal-trashcan.png)

跳到[作业 2](../Assignment02/README.md) 。
