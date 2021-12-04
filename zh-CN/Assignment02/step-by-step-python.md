# 作业 2 - 添加 Dapr 服务到服务调用

## 作业目标

To complete this assignment, you must reach the following goals:

- VehicleRegistrationService 和 FineCollectionService 都使用 Dapr sidecar 运行。
- The FineCollectionService uses the Dapr service invocation building block to call the `/vehicleinfo/{licensenumber}` endpoint on the VehicleRegistrationService.

This assignment targets number **1** in the end-state setup:

<img src="https://github.com/dotnet-cn/dapr-workshop/blob/main/zh-CN/img/dapr-setup.png?raw=true" class="">

## 第 1 步：使用 Dapr 启动 VehicleRegistrationService

In assignment 1, you started all the services using `uvicorn <app>`. When you want to run a service with a Dapr sidecar that handles its communication, you need to start it using the Dapr CLI. There are a couple of things you need to specify when starting the service:

- The service needs a unique id which Dapr can use to find it. This is called the *app-id* (or application Id). You specify this with the `--app-id` flag on the command-line.

- Each of the services listens on a different HTTP port for requests (to prevent port collisions on localhost). The VehicleRegistrationService runs on port `6002` for instance. You need to tell Dapr this port so the Dapr sidecar can communicate with the service. You specify this with the `--app-port` flag on the command-line.

- The service can use HTTP or gRPC to communicate with the Dapr sidecar. By default these ports are `3500` and `50001`. But to prevent confusion, you'll use totally different port numbers in the assignments. To prevent port collisions on the local machine when running multiple services, you have to specify a unique HTTP and gRPC port per service. You specify this with the `--dapr-http-port` and `--dapr-grpc-port` flags on the command-line. Throughout the workshop, you will use the following ports:

    服务 | 应用端口 | Dapr sidecar HTTP 端口 | Dapr sidecar gRPC 端口
    --- | --- | --- | ---
    TrafficControlService | 6000 | 3600 | 60000
    FineCollectionService | 6001 | 3601 | 60001
    VehicleRegistrationService | 6002 | 3602 | 60002

- Finally you need to tell Dapr how to start the service. The services are Python services which can be started with the command `uvicorn <app>`.

您将使用`run`命令并在命令行上指定上述所有选项：

1. 确保你已经在你的机器上启动了 Docker Desktop，并且安装了 Dapr CLI 和运行时（参见[先决条件](../README.md#prerequisites)）。

2. 在 VS Code 中打开源代码文件夹。

3. 在 VS Code 中打开终端窗口并确保当前文件夹是`VehicleRegistrationService` 。

4. 输入以下命令以运行带有 Dapr sidecar 的 VehicleRegistrationService：

    ```console
    dapr run --app-id vehicleregistrationservice --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 -- uvicorn vehicle_registration:app --port 6002
    ```

5. 检查日志是否有任何错误。如您所见，Dapr 和应用程序日志都显示为输出。

Now you're running a 'Daprized' version of the VehicleRegistrationService. As you might have noticed, you didn't need to change any code for this to work. The VehicleRegistrationService is still just a web API listening for requests. Only now, you've started it with a Dapr sidecar next to it that can communicate with it. This means other services can use Dapr to call this service. This is what you'll do in the next step.

## 步骤 2：使用 Dapr 服务调用调用 VehicleRegistrationService

在此步骤中，您将更改 FineCollectionService 的代码，使其使用 Dapr 服务调用来调用 VehicleRegistrationService。

首先，您将更改代码，使其调用 Dapr sidecar：

1. 在 VS Code 中打开文件`FineCollectionService/fine_collection/clients.py`

2. 检查`get_vehicle_info`方法。它包含对 VehicleRegistrationService 的调用以检索车辆信息：

    ```python
    response = requests.get(f"{self.base_address}/vehicleinfo/{license_number}")

    if response.status_code == 200:
       return Vehicle.parse_raw(response.content)
    ```

    `requests`来调用 VehicleRegistrationService。其使用 API 的基址由`settings.py`确定，后者`.env`存储在`FineCollectionService`的 .env 文件中读取设置。

3. 在 VS Code 中打开文件`FineCollectionService/.env`

    在这里，我们看到正在配置的实际值。检查`VEHICLE_REGISTRATION_ADDRESS`设置。您可以看到在 HTTP 调用中，使用了 VehicleRegistrationService（在端口 6002 上运行）的 URL。

4. 在 Dapr sidecar 上调用 Dapr 服务调用构建块的 API 是：

    ```http
    http://localhost:<daprPort>/v1.0/invoke/<appId>/method/<method-name>
    ```

    您可以使用适当的值替换此 URL 中的占位符，以便 FineCollectionService 的 sidecar 可以调用 VehicleRegistrationService，这将生成以下 URL：

    ```http
    http://localhost:3601/v1.0/invoke/vehicleregistrationservice/method/vehicleinfo/{licenseNumber}
    ```

    正如您在此 URL 中看到的，FineCollectionService 的 Dapr sidecar 将在 HTTP 端口`3601`上运行。

5. 将配置文件中的 URL 替换为新的 Dapr 服务调用 URL。该行现在应如下所示：

    ```sh
    VEHICLE_REGISTRATION_ADDRESS=http://localhost:3601/v1.0/invoke/vehicleregistrationservice/method
    ```

    设置文件中的根 URL 被替换为从步骤 3 到 URL 的`<method-name>`部分的占位符 URL。

    > It's important to really grasp the sidecar pattern used by Dapr. In this case, the FineCollectionService calls the VehicleRegistrationService by **calling its own Dapr sidecar**! The FineCollectionService doesn't need to know anymore where the VehicleRegistrationService lives because its Dapr sidecar will take care of that. It will find it based on the `app-id` specified in the URL and call the target service's sidecar.

6. 在 VS Code 中打开一个**新的**终端窗口并确保当前文件夹是`FineCollectionService` 。

7. 输入以下命令以使用 Dapr sidecar 运行 FineCollectionService：

    ```console
    dapr run --app-id finecollectionservice --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 -- uvicorn fine_collection:app --port 6001
    ```

8. 检查日志是否有任何错误。如您所见，Dapr 和应用程序日志都显示为输出。

现在您要测试应用程序：

1. 在 VS Code 中打开一个**新的**终端窗口并将当前文件夹更改为`TrafficControlService` 。

2. 输入以下命令以运行 TrafficControlService：

    ```console
    uvicorn traffic_control:app --port 6000
    ```

> 在此分配中，TrafficControlService 不需要与 Dapr sidecar 一起运行。这是因为它仍然会像以前一样通过 HTTP 调用 FineCollectionService。

The services are up &amp; running. Now you're going to test this using the simulation.

1. 在 VS Code 中打开一个**新的**终端窗口并将当前文件夹更改为`Simulation` 。

2. 开始模拟：

    ```console
    python3 simulation
    ```

You should see similar logging as before when you ran the application. So all the functionality works the same, but now you use Dapr service invocation to communicate between the FineCollectionService and the VehicleRegistrationService.

## 第 3 步：将 Dapr 服务调用与 Dapr SDK for Python 结合使用

在此步骤中，您将更改 FineCollectionService 的代码，使其使用 Dapr SDK for Python 来调用 VehicleRegistrationService。 SDK 提供了一种更集成的方式来调用 Dapr sidecar API。

首先停止模拟：

1. 在 VS Code 中打开运行 Camera Simulation 的终端窗口。

2. `Ctrl-C`停止模拟，然后单击标题栏中的垃圾桶图标（或键入`exit`命令）关闭终端窗口。

3. 在 VS Code 中打开运行 FineCollectionService 的终端窗口。

4. `Ctrl-C`停止服务。保持此终端窗口打开并聚焦。

5. 运行以下命令为 Python 安装 Dapr SDK：

    ```console
    pip3 install dapr
    ```

    > 适用于 Python 的 Dapr SDK 包含我们将用于直接调用 Dapr API `DaprClient` SDK 中有更多组件。我们稍后将在研讨会中使用它们。

现在您将更改代码以使用 Dapr 提供的`DaprClient`来调用 VehicleRegistrationService。在第 2 步中，我们使用`requests`库让我们的应用程序不知道 Dapr。在这一步中，我们将升级我们的应用程序以使用 SDK。

1. 在 VS Code 中打开文件`FineCollectionService/fine_collection/clients.py`

2. 将文件内容替换为以下代码：

    ```python
    from pydantic import BaseModel
    from dapr.clients import DaprClient


    class Vehicle(BaseModel):
       vehicleId: str
       make: str
       model: str
       ownerName: str
       ownerEmail: str


    class VehicleRegistrationClient:
       def __init__(self, base_address: str):
          self.base_address = base_address

       def get_vehicle_info(self, license_number: str) -> Vehicle:
          with DaprClient() as client:
                response = client.invoke_method(
                   "vehicleregistrationservice",
                   f"vehicleinfo/{license_number}",
                   data=b'',
                   http_verb="GET"
                )

                return Vehicle.parse_raw(response.text())

    ```

现在将 FineCollectionService 更改为使用 Dapr SDK 进行服务调用。让我们测试一下。

1. 如果您按照此作业中的说明进行操作，则 VehicleRegistrationService 和 TrafficControlService 仍在运行。

2. 在运行 FineCollectionService 的 VS Code 中打开终端窗口。

3. 输入以下命令再次启动更改后的 FineCollectionService：

    ```console
    dapr run --app-id finecollectionservice --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 -- uvicorn fine_collection:app --port 6001
    ```

The services are up &amp; running. Now you're going to test this using the simulation.

1. 在 VS Code 中打开一个**新的**终端窗口并将当前文件夹更改为`Simulation` 。

2. 开始模拟：

    ```console
    python3 simulation
    ```

运行应用程序时，您应该会看到与之前类似的日志记录。

## 第 4 步：使用 Dapr 可观察性

So how can you check whether or not the call to the VehicleRegistrationService is handled by Dapr? Well, Dapr has some observability built in. You can look at Dapr traffic using Zipkin:

1. 打开浏览器并访问此 URL： [http://localhost:9411/zipkin](http://localhost:9411/zipkin) 。

2. 单击屏幕右上角的`RUN QUERY`按钮以搜索跟踪。

3. You should see the calls between the FineCollectionService and the VehicleRegistrationService. You can expand and collapse each trace and click the `SHOW` button to get more details:

    ![](img/zipkin-traces.png)

4. If you click the dependencies button and search, you will see the services and the traffic flowing between them:

    ![](img/zipkin-dependencies.gif)

## 下一个任务

在继续下一个任务之前，请确保停止所有正在运行的进程并关闭 VS Code 中的所有终端窗口。

转到[作业 3](../Assignment03/README.md) 。
