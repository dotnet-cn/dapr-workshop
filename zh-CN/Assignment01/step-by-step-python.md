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

4. 运行命令`pip3 install -r requirements.txt`

5. 使用`uvicorn vehicle_registration:app --port 6002`启动服务

> **注意**，如果您在 Linux/Ubuntu 上运行，则`uvicorn`可能无法执行。您可以运行命令`sudo apt install uvicorn`使`uvicorn`在系统中可用。

> 如果您在此处收到错误消息，请仔细检查您是否已安装工作坊的[所有预装软件！](../README.md#Prerequisites)

现在您可以测试是否可以调用 VehicleRegistrationService。您可以使用浏览器、cURL 或其他一些 HTTP 客户端来执行此操作。但是有一种直接从 VS Code 测试 RESTful API 的便捷方法（使用 VS Code的REST Client扩展 ）：

1. 在编辑器中打开文件`VehicleRegistrationService/test.http`。此文件中的请求模拟获得某个车牌号的车辆和所有者信息。

2. 在文件中点击`Send request`发送到API的请求：

    ![REST client](img/rest-client.png)

3. 请求的响应将显示在右侧的单独窗口中。它应该是一个带有 HTTP 状态码`200 OK`的响应，并且正文应该包含一些随机的车辆和所有者信息：

    ```json
    HTTP/1.1 200
    Connection: keep-alive
    Content-Type: application/json
    Date: Wed, 16 Jun 2021 19:39:05 GMT
    Keep-Alive: timeout=60
    Transfer-Encoding: chunked

    {
        "vehicle_id": "KZ-49-VX",
        "brand": "Toyota",
        "model": "Rav 4",
        "owner_name": "Angelena Fairbairn",
        "owner_email": "angelena.fairbairn@outlook.com"
    }
    ```

4. 检查终端窗口中的日志记录。它应该是这样的：

    ```console
    > uvicorn vehicle_registration:app --port 6002
    INFO:     Started server process [29384]
    INFO:     Waiting for application startup.
    INFO:     Application startup complete.
    INFO:     Uvicorn running on http://127.0.0.1:6002 (Press CTRL+C to quit)
    INFO:     127.0.0.1:64855 - "GET /vehicleinfo/KZ-49-VX HTTP/1.1" 200 OK
    ```

## 步骤 2. 运行 FineCollection 服务

1. 确保 VehicleRegistrationService 服务正在运行（步骤 1 的结果）。

2. 在 VS Code 中打开一个**新的终端窗口。**

    > 您可以使用热键做到这一点，在Windows上使用 `Ctrl-``，在MacOS使用`Shift-Ctrl-``，或点击终端窗口的标题栏中的`+`按钮：<br><img alt="">

3. 确保当前文件夹是`FineCollectionService` 。

4. 运行命令`pip3 install -r requirements.txt`

5. 使用`uvicorn fine_collection:app --port 6001`启动服务。

6. 在 VS Code 中打开文件`FineCollectionService/test.http`。此文件中的请求模拟向 FineCollectionService 发送检测到的超速违规。

7. 单击`Execute request`以向 API 发送请求。

8. 请求的响应将显示在右侧的单独窗口中。它应该是 HTTP 状态码`200 OK`且没有正文的响应。

9. 检查终端窗口中的日志记录。它应该是这样的：

    ```console
    ❯ uvicorn fine_collection:app --port 6001
    INFO:     Started server process [3108]
    INFO:     Waiting for application startup.
    INFO:     Application startup complete.
    INFO:     Uvicorn running on http://127.0.0.1:6001 (Press CTRL+C to quit)
    INFO:     127.0.0.1:52778 - "POST /collectfine HTTP/1.1" 200 OK
    ```

## 步骤 3. 运行 TrafficControl 服务

1. 确保 VehicleRegistrationService 和 FineCollectionService 正在运行（步骤 1 和 2 的结果）。

2. 在 VS Code 中打开一个**新的**终端窗口并确保当前文件夹是`TrafficControlService` 。

3. 运行命令`pip3 install -r requirements.txt`

4. 使用`uvicorn traffic_control:app --port 6000`启动服务。

5. 在 VS Code 中打开`TrafficControlService/test.http`

6. 单击`Execute request`以向 API 发送两个请求。

7. 请求的响应将显示在右侧的单独窗口中。两个请求都应产生 HTTP 状态码`200 OK`且没有正文的响应。

8. 检查终端窗口中的日志记录。它应该是这样的：

    ```console
    ❯ uvicorn traffic_control:app --port 6000
    INFO:     Started server process [29680]
    INFO:     Waiting for application startup.
    INFO:     Application startup complete.
    INFO:     Uvicorn running on http://127.0.0.1:6000 (Press CTRL+C to quit)
    INFO:     127.0.0.1:56477 - "POST /entrycam HTTP/1.1" 200 OK
    Vehicle XT-346-Y is over the speed limit. Collecting fine
    INFO:     127.0.0.1:56478 - "POST /exitcam HTTP/1.1" 200 OK
    ```

9. 还要检查 FineCollectionService 的日志记录。

    > 您可以通过使用终端窗口标题栏中的下拉菜单选择另一个终端窗口来执行此操作：![](img/terminal-dropdown.png)

    您应该会看到 FineCollectionService 正在处理的超速违规：

    ```console
    ❯ uvicorn fine_collection:app --port 6001
    INFO:     Started server process [8728]
    INFO:     Waiting for application startup.
    INFO:     Application startup complete.
    INFO:     Uvicorn running on http://127.0.0.1:6001 (Press CTRL+C to quit)
    INFO:     127.0.0.1:49916 - "POST /collectfine HTTP/1.1" 200 OK
    ```

## 步骤 4. 运行模拟

您已经使用 REST Client直接测试了 API。现在您将运行的模拟是实际模拟汽车在高速公路上行驶的，该模拟将模拟 3 个入口和出口的摄像头（每个车道一个）。

1. 在 VS Code 中打开一个新的终端窗口，并确保当前文件夹是`Simulation` 。

2. 运行命令`pip3 install -r requirements.txt`

3. 使用`python3 simulation`启动服务。

4. 在模拟窗口中，您应该看到如下内容：

    ```console
    ❯ python simulation
    Starting agent 0...
    Starting agent 1...
    Starting agent 2...
    Simulation started. Press Ctrl+C to exit.
    Sent car 20-TK-YH into the traffic control system
    Sent car ZD-84-GG into the traffic control system
    Sent car PZ-36-FF into the traffic control system
    ```

5. 还要检查所有其他终端窗口中的日志记录。您应该在日志记录中看到检测到的所有进入和退出事件以及任何超速违规行为。

现在我们知道应用程序运行正常。是时候开始将 Dapr 添加到应用程序中了。

## 下一个任务

在继续下一个任务之前，请确保停止所有正在运行的进程并关闭 VS Code 中的所有终端窗口。通过在终端窗口中按`Ctrl-C`来停止服务或模拟。要关闭终端窗口，请输入`exit`命令。

> 您可以通过单击标题栏中的垃圾桶图标来快速关闭终端窗口：![](img/terminal-trashcan.png)

转到[作业 2](../Assignment02/README.md) 。
