# 作业 1 - 运行应用程序

在本作业中，您将运行应用程序以确保一切正常工作。

## 作业目标

要完成此任务，您必须达到以下目标：

- 所有服务都在运行。
- 日志记录表明所有服务都在正常工作。

提醒一下，这就是服务之间的交互方式：

<img src="../img/services.png" style="zoom: 67%;">

## 步骤 1. 运行 VehicleRegistration 服务

1. 在 VS Code 中打开源代码文件夹。

    > 在整个作业中，您需要在同一 VS Code 实例中执行所有步骤。

2. 在 VS Code 中打开终端窗口。

    > 您可以使用热键`Ctrl-`` (Windows) 或`Shift-Ctrl-`` (macOS) 执行此操作。

3. 确保当前文件夹是`VehicleRegistrationService` 。

4. 使用`dotnet run`启动服务。

> 如果您在此处收到错误消息，请仔细检查您是否已安装工作坊的[所有预装软件！](../README.md#Prerequisites)

现在您可以测试是否可以调用 VehicleRegistrationService。您可以使用浏览器、cURL 或其他一些 HTTP 客户端来执行此操作。但是有一种直接从 VS Code 测试 RESTful API 的便捷方法（使用 VS Code的REST Client扩展 ）：

1. 在 VS Code 中打开文件`VehicleRegistrationService/test.http`。此文件中的请求模拟获得某个车牌号的车辆和所有者信息。

2. 在文件中点击`Send request`发送到API的请求：

    ![REST客户端](img/rest-client.png)

3. 请求的响应将显示在右侧的单独窗口中。它应该是一个带有 HTTP 状态码`200 OK`的响应，并且正文应该包含一些随机的车辆和所有者信息：

    ```json
    HTTP/1.1 200 OK
    Connection: close
    Date: Mon, 01 Mar 2021 07:15:55 GMT
    Content-Type: application/json; charset=utf-8
    Server: Kestrel
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
    ❯ dotnet run
    info: Microsoft.Hosting.Lifetime[0]
          Now listening on: http://localhost:6002
    info: Microsoft.Hosting.Lifetime[0]
          Application started. Press Ctrl+C to shut down.
    info: Microsoft.Hosting.Lifetime[0]
          Hosting environment: Development
    info: Microsoft.Hosting.Lifetime[0]
          Content root path: D:\dev\Dapr\dapr-workshop\dotnet\VehicleRegistrationService
    info: VehicleRegistrationService.Controllers.VehicleInfoController[0]
          Retrieving vehicle-info for licensenumber KZ-49-VX
    ```

## 步骤 2. 运行 FineCollection 服务

1. 确保 VehicleRegistrationService 服务正在运行（步骤 1 的结果）。

2. 在 VS Code 中打开一个**新的终端窗口。**

    > 您可以使用热键做到这一点，在Windows上使用 `Ctrl-``，在MacOS使用`Shift-Ctrl-``，或点击终端窗口的标题栏中的`+`按钮：<br>![](img/terminal-new.png)

3. 确保当前文件夹是`FineCollectionService` 。

4. 使用`dotnet run`启动服务。

5. 在 VS Code 中打开文件`FineCollectionService/test.http`。此文件中的请求模拟向 FineCollectionService 发送检测到的超速违规。

6. 单击`Execute request`以向 API 发送请求。

7. 请求的响应将显示在右侧的单独窗口中。它应该是 HTTP 状态码`200 OK`且没有正文的响应。

8. 检查终端窗口中的日志记录。它应该是这样的：

    ```console
    ❯ dotnet run
    info: Microsoft.Hosting.Lifetime[0]
          Now listening on: http://localhost:6001
    info: Microsoft.Hosting.Lifetime[0]
          Application started. Press Ctrl+C to shut down.
    info: Microsoft.Hosting.Lifetime[0]
          Hosting environment: Development
    info: Microsoft.Hosting.Lifetime[0]
          Content root path: D:\dev\Dapr\dapr-workshop\dotnet\FineCollectionService
    info: System.Net.Http.HttpClient.Default.LogicalHandler[100]
          Start processing HTTP request GET http://localhost:6002/vehicleinfo/RT-318-K
    info: System.Net.Http.HttpClient.Default.ClientHandler[100]
          Sending HTTP request GET http://localhost:6002/vehicleinfo/RT-318-K
    info: System.Net.Http.HttpClient.Default.ClientHandler[101]
          Received HTTP response headers after 148.5761ms - 200
    info: System.Net.Http.HttpClient.Default.LogicalHandler[101]
          End processing HTTP request after 156.5675ms - 200
    info: FineCollectionService.Controllers.CollectionController[0]
          Sent speeding ticket to Cassi Dakes. Road: A12, Licensenumber: RT-318-K, Vehicle: Mercedes SLK, Violation: 15 Km/h, Fine: 130 Euro, On: 20-09-2020 at 08:33:41.
    ```

## 步骤 3. 运行 TrafficControl 服务

1. 确保 VehicleRegistrationService 和 FineCollectionService 正在运行（步骤 1 和 2 的结果）。

2. 在 VS Code 中打开一个**新的**终端窗口并确保当前文件夹是`TrafficControlService` 。

3. 使用`dotnet run`启动服务。

4. 在 VS Code 中打开`test.http`文件

5. 单击文件中的`Execute request`以向 API 发送两个请求。

6. 请求的响应将显示在右侧的单独窗口中。两个请求都应产生 HTTP 状态码`200 OK`且没有正文的响应。

7. 检查终端窗口中的日志记录。它应该是这样的：

    ```console
    ❯ dotnet run
    info: Microsoft.Hosting.Lifetime[0]
          Now listening on: http://localhost:6000
    info: Microsoft.Hosting.Lifetime[0]
          Application started. Press Ctrl+C to shut down.
    info: Microsoft.Hosting.Lifetime[0]
          Hosting environment: Development
    info: Microsoft.Hosting.Lifetime[0]
          Content root path: D:\dev\Dapr\dapr-workshop\dotnet\TrafficControlService
    info: TrafficControlService.Controllers.TrafficController[0]
          ENTRY detected in lane 1 at 10:38:47 of vehicle with license-number XT-346-Y.
    info: TrafficControlService.Controllers.TrafficController[0]
          EXIT detected in lane 1 at 10:38:52 of vehicle with license-number XT-346-Y.
    info: TrafficControlService.Controllers.TrafficController[0]
          Speeding violation detected (15 KMh) of vehiclewith license-number XT-346-Y.
    info: System.Net.Http.HttpClient.Default.LogicalHandler[100]
          Start processing HTTP request POST http://localhost:6001/collectfine
    info: System.Net.Http.HttpClient.Default.ClientHandler[100]
          Sending HTTP request POST http://localhost:6001/collectfine
    info: System.Net.Http.HttpClient.Default.ClientHandler[101]
          Received HTTP response headers after 254.3259ms - 200
    info: System.Net.Http.HttpClient.Default.LogicalHandler[101]
          End processing HTTP request after 264.8462ms - 200
    ```

8. 还要检查 FineCollectionService 的日志记录。

    > 您可以通过在终端窗口右侧的**选项卡视图中**单击来选择另一个终端窗口：<br>![](img/terminal-tabs.png)

    您应该会看到 FineCollectionService 正在处理的超速违规：

    ```console
    info: System.Net.Http.HttpClient.Default.LogicalHandler[100]
          Start processing HTTP request GET http://localhost:6002/vehicleinfo/XT-346-Y
    info: System.Net.Http.HttpClient.Default.ClientHandler[100]
          Sending HTTP request GET http://localhost:6002/vehicleinfo/XT-346-Y
    info: System.Net.Http.HttpClient.Default.ClientHandler[101]
          Received HTTP response headers after 175.9129ms - 200
    info: System.Net.Http.HttpClient.Default.LogicalHandler[101]
          End processing HTTP request after 176.0221ms - 200
    info: FineCollectionService.Controllers.CollectionController[0]
          Sent speeding ticket to Refugio Petterson. Road: A12, Licensenumber: XT-346-Y, Vehicle: Fiat Panda, Violation: 15 Km/h, Fine: 130 Euro, On: 10-09-2020 at 10:38:52.
    ```

## 步骤 4. 运行模拟

您已经使用 REST Client直接测试了 API。现在您将运行的模拟是实际模拟汽车在高速公路上行驶的，该模拟将模拟 3 个入口和出口的摄像头（每个车道一个）。

1. 在 VS Code 中打开一个新的终端窗口，并确保当前文件夹是`Simulation` 。

2. 使用`dotnet run`启动服务。

3. 在模拟窗口中，您应该看到如下内容：

    ```console
    ❯ dotnet run
    Start camera 2 simulation.
    Start camera 3 simulation.
    Start camera 1 simulation.
    Simulated ENTRY of vehicle with license-number Y-373-JF in lane 2
    Simulated ENTRY of vehicle with license-number RL-001-D in lane 2
    Simulated ENTRY of vehicle with license-number TF-352-N in lane 3
    Simulated ENTRY of vehicle with license-number JY-94-SY in lane 1
    Simulated ENTRY of vehicle with license-number 4-JSL-09 in lane 1
    Simulated ENTRY of vehicle with license-number 87-DGR-7 in lane 3
    Simulated ENTRY of vehicle with license-number 44-FK-64 in lane 2
    Simulated EXIT of vehicle with license-number Y-373-JF in lane 3
    Simulated ENTRY of vehicle with license-number TH-822-X in lane 1
    Simulated ENTRY of vehicle with license-number G-127-SN in lane 2
    Simulated ENTRY of vehicle with license-number 44-TN-JD in lane 3
    Simulated ENTRY of vehicle with license-number T-252-NJ in lane 2
    Simulated ENTRY of vehicle with license-number 1-HXS-04 in lane 2
    Simulated EXIT of vehicle with license-number RL-001-D in lane 1
    Simulated ENTRY of vehicle with license-number DJ-940-S in lane 1
    ```

4. 还要检查所有其他终端窗口中的日志记录。您应该在日志记录中看到检测到的所有进入和退出事件以及任何超速违规行为。

现在我们知道应用程序运行正常。是时候开始将 Dapr 添加到应用程序中了。

## 下一个任务

在继续下一个任务之前，请确保停止所有正在运行的进程并关闭 VS Code 中的所有终端窗口。通过在终端窗口中按`Ctrl-C`来停止服务或模拟。要关闭终端窗口，请输入`exit`命令。

> 您可以通过单击终端窗口右侧**选项卡视图**中终端实例旁边的垃圾桶图标来快速关闭终端窗口：
>
> ![](img/terminal-trashcan.png)
>
> 如果您只打开了 1 个终端窗口，则在标题栏中点击即可：
>
> ![](img/terminal-trashcan-titlebar.png)

转到[作业 2](../Assignment02/README.md) 。
