# 作业 2 - 添加 Dapr 服务到服务调用

## 作业目标

要完成此作业，您必须达到以下目标：

- VehicleRegistrationService 和 FineCollectionService 都使用 Dapr sidecar 运行。
- FineCollectionService 使用 Dapr 服务调用构建块来调用`/vehicleinfo/{licensenumber}`终端。

此作业是end-state设置中的**第 1 项：**

<img src="../img/dapr-setup.png" style="zoom: 67%;">

## 第 1 步：使用 Dapr 启动 VehicleRegistrationService

在作业 1 中，您使用`dotnet run`启动了所有服务。当您想要使用用于通信的 Dapr sidecar 运行服务时，您需要使用 Dapr CLI 启动它。在启动服务时，您需要注意以下几点：

- 该服务需要一个唯一的 ID，Dapr 可以使用该ID来找到服务，我们把它称为*app-id* （或应用程序 ID）。您可以在命令行中输入`--app-id`

- 每个服务在不同的 HTTP 端口上侦听请求（以防止本地主机上的端口冲突）。例如， VehicleRegistrationService 在端口`6002`上运行。您需要告诉 Dapr 这个端口，以便 Dapr sidecar 可以与服务通信。您可以在命令行中使用`--app-port`

- 该服务可以使用 HTTP 或 gRPC 与 Dapr sidecar 进行通信。默认情况下，这些端口是`3500`和`50001` 。但是为了防止混淆，您将使用完全不同的端口号。为了在运行多个服务时防止本地机器上的端口冲突，您必须为每个服务指定唯一的 HTTP 和 gRPC 端口，可以在命令行中使用`--dapr-http-port`和`--dapr-grpc-port`。您将使用以下端口：

    服务 | 应用端口 | Dapr sidecar HTTP 端口 | Dapr sidecar gRPC 端口
    --- | --- | --- | ---
    交通控制服务 | 6000 | 3600 | 60000
    精品收藏服务 | 6001 | 3601 | 60001
    车辆登记服务 | 6002 | 3602 | 60002

- 最后你需要告诉 Dapr 如何启动服务，这些服务是 .NET 服务，可以使用`dotnet run`命令启动。

您将使用`run`命令并在命令行上指定上述所有选项：

1. 确保你已经在你的机器上启动了 Docker Desktop，并且安装了 Dapr CLI 和Dapr运行时（参见[先决条件](../README.md#prerequisites)）。

2. 在 VS Code 中打开源代码文件夹。

3. 在 VS Code 中打开终端窗口并确保当前文件夹是`VehicleRegistrationService` 。

4. 输入以下命令以运行带有 Dapr sidecar 的 VehicleRegistrationService：

    ```console
    dapr run --app-id vehicleregistrationservice --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 dotnet run
    ```

5. 检查日志是否有任何错误。如您所见，Dapr 和应用程序日志都都会输出。

现在您正在运行 VehicleRegistrationService 的“Daprized”版本，您可能已经注意到，您无需更改任何代码即可使其正常工作。 VehicleRegistrationService 仍然只是一个侦听请求的 Web API。直到现在，您才开始使用它的 Dapr sidecar，服务可以与之通信，这意味着其他服务可以使用 Dapr 来调用此服务，这也是您将在下一步中做的操作。

## 步骤 2：使用 Dapr 服务调用 调用 VehicleRegistrationService

在此步骤中，您将更改 FineCollectionService 的代码，使其使用 Dapr 服务调用来调用 VehicleRegistrationService。

首先，您将更改代码，使其调用 Dapr sidecar：

1. 在 VS Code 中打开文件`FineCollectionService/Controllers/CollectionController.cs`

2. 检查`CollectFine`方法。它包含对 VehicleRegistrationService 的调用以获取车辆信息：

    ```csharp
    // get owner info
    var vehicleInfo = await _vehicleRegistrationService.GetVehicleInfo(speedingViolation.VehicleId);
    ```

    `_vehicleRegistrationService`是使用 .NET `HttpClient`调用VehicleRegistrationService 的代理实例。您将更改该代理，使其使用 Dapr 服务调用。

3. 在 VS Code 中打开文件`FineCollectionService/Proxies/VehicleRegistrationService.cs`

4. 检查`GetVehicleInfo`方法。您可以看到在 HTTP 调用中，使用了 VehicleRegistrationService（在端口 6002 上运行）的 URL。

5. 在 Dapr sidecar 上调用 Dapr 服务调用构建块的 API 是：

    ```http
    http://localhost:<daprPort>/v1.0/invoke/<appId>/method/<method-name>
    ```

    您可以使用适当的值替换此 URL 中的占位符，以便 FineCollectionService 的 sidecar 可以调用 VehicleRegistrationService，这将生成以下 URL：

    ```http
    http://localhost:3601/v1.0/invoke/vehicleregistrationservice/method/vehicleinfo/{licenseNumber}
    ```

    正如在此 URL 中看到的，FineCollectionService 的 Dapr sidecar 将在 HTTP 端口`3601`上运行。

6. 将代码中的 URL 替换为新的 Dapr 服务调用 URL，现在代码应该是这样的：

    ```csharp
    public async Task<VehicleInfo> GetVehicleInfo(string licenseNumber)
    {
        return await _httpClient.GetFromJsonAsync<VehicleInfo>(
            $"http://localhost:3601/v1.0/invoke/vehicleregistrationservice/method/vehicleinfo/{licenseNumber}");
    }
    ```

    > 真正掌握 Dapr 使用的 sidecar 模式很重要。在这种情况下，FineCollectionService 通过**调用自己的 Dapr sidecar**来调用 VehicleRegistrationService！ FineCollectionService 不再需要知道 VehicleRegistrationService 所在的位置，因为它的 Dapr sidecar 会处理这些，它会根据`app-id`找到它并调用目标服务的 sidecar。

7. 在 VS Code 中打开一个**新的**终端窗口并确保当前文件夹是`FineCollectionService` 。

8. 通过构建代码检查所有代码更改是否正确：

    ```console
    dotnet build
    ```

    如果您看到任何警告或错误，请查看前面的步骤以确保代码正确。

9. 输入以下命令以使用 Dapr sidecar 运行 FineCollectionService：

    ```console
    dapr run --app-id finecollectionservice --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 dotnet run
    ```

10. 检查日志是否有任何错误。如您所见，Dapr 和应用程序日志都显示为输出。

现在您要测试应用程序：

1. 在 VS Code 中打开一个**新的**终端窗口并将当前文件夹更改为`TrafficControlService` 。

2. 输入以下命令以运行 TrafficControlService：

3. ```console
    dotnet run
    ```

> 在此作业中，TrafficControlService 不需要与 Dapr sidecar 一起运行。这是因为它仍然会像以前一样通过 HTTP 调用 FineCollectionService。

服务已启动并正在运行。现在您将使用模拟来测试它。

1. 在 VS Code 中打开一个**新的**终端窗口并将当前文件夹更改为`Simulation` 。

2. 开始模拟：

    ```console
    dotnet run
    ```

运行应用程序时，您应该会看到与之前类似的日志记录，所有功能都一样，但现在使用 Dapr 服务调用在 FineCollectionService 和 VehicleRegistrationService 之间进行通信。

## 第 3 步：将 Dapr 服务调用与 Dapr SDK for  .NET 结合使用

在此步骤中，您将更改 FineCollectionService 的代码，使其使用 Dapr SDK for .NET 来调用 VehicleRegistrationService。 SDK 提供了一种更集成的方式来调用 Dapr sidecar API。

首先停止模拟：

1. 在 VS Code 中打开运行 Camera Simulation 的终端窗口。

2. `Ctrl-C`停止模拟，然后单击标题栏中的垃圾桶图标（或键入`exit`命令）关闭终端窗口。

3. 在 VS Code 中打开运行 FineCollectionService 的终端窗口。

4. `Ctrl-C`停止服务。保持此终端窗口打开并聚焦。

5. 添加对 Dapr ASP.NET Core 集成库的引用：

    ```console
    dotnet add package Dapr.AspNetCore -v 1.4.0
    ```

    > `Dapr.AspNetCore`包 包含`DaprClient`类以及与 ASP.NET Core 的其他集成。因为这些服务都是 ASP.NET Core Web API，所以我们将在整个工作坊过程中使用这个包。

现在，更改代码以使用 Dapr SDK `HttpClient`集成来调用 VehicleRegistrationService。 `HttpClient`集成使用常规`HttpClient`进行服务调用，而 SDK 确保调用通过 Dapr sidecar 进行路由。

1. 在 VS Code 中打开文件`FineCollectionService/Startup.cs`

2. 在此文件中添加 using 语句以确保您可以使用 Dapr 客户端：

    ```csharp
    using Dapr.Client;
    ```

3. `ConfigureServices`方法包含这两行代码，它们使用依赖注入`HttpClient`和`VehicleRegistrationService`代理（使用`HttpClient`

    ```csharp
    // 添加服务代理
    services.AddHttpClient();
    services.AddSingleton<VehicleRegistrationService>();
    ```

4. 用以下几行替换这两行：

    ```csharp
    // 添加服务代理
    services.AddSingleton<VehicleRegistrationService>(_ =>
        new VehicleRegistrationService(DaprClient.CreateInvokeHttpClient(
            "vehicleregistrationservice", "http://localhost:3601")));
    ```

    正如在此代码段中看到的，使用`DaprClient`创建一个`HttpClient`实例来执行服务调用。您使用`app-id`指定要与之通信的服务，还要指定 Dapr sidecar 的地址，因为 FineCollectionService 的 sidecar 不使用默认的 Dapr HTTP 端口（3500）。生成的`HttpClient`实例显式传入`VehicleRegistrationService`代理的构造函数中。

    > 在使用`Dapr.AspNetCore`库时 ，这是Dapr 与 ASP.NET Core 深度集成的示例。您仍然可以使用`HttpClient` （及其丰富的功能集），但在后台使用 Dapr 服务调用构建块。

5. 在 VS Code 中打开文件`FineCollectionService/Proxies/VehicleRegistrationService.cs`

6. 因为传入这个类的`HttpClient`就是为某个 `app-id`创建的，所以可以省略请求 URL 中的主机信息， `GetVehicleInfo`使用的 URL 更改为`/vehicleinfo/{license-number}` 。该方法现在应如下所示：

    ```csharp
    public async Task<VehicleInfo> GetVehicleInfo(string licenseNumber)
    {
        return await _httpClient.GetFromJsonAsync<VehicleInfo>(
            $"/vehicleinfo/{licenseNumber}");
    }
    ```

现在将 FineCollectionService 更改为使用 Dapr SDK 进行服务调用，让我们测试一下。

1. 如果您按照此作业中的说明进行操作，则 VehicleRegistrationService 和 TrafficControlService 仍在运行。

2. 在运行 FineCollectionService 的 VS Code 中打开终端窗口。

3. 输入以下命令再次启动更改后的 FineCollectionService：

    ```console
    dapr run --app-id finecollectionservice --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 dotnet run
    ```

服务已启动并正在运行。现在将使用模拟来测试它。

1. 在 VS Code 中打开一个**新的**终端窗口并将当前文件夹更改为`Simulation` 。

2. 开始模拟：

    ```console
    dotnet run
    ```

运行应用程序时，您应该会看到与之前类似的日志记录。

## 第 4 步：使用 Dapr 可观察性

那么如何检查对 VehicleRegistrationService 的调用是否由 Dapr 处理呢？Dapr 内置了一些可观察性，您可以使用 Zipkin 查看 Dapr 流量：

1. 打开浏览器并访问此 URL： [http://localhost:9411/zipkin](http://localhost:9411/zipkin) 。

2. 单击屏幕右上角的`RUN QUERY`按钮以搜索跟踪。

3. 您应该会看到 FineCollectionService 和 VehicleRegistrationService 之间的调用，可以展开和折叠每条跟踪，然后单击“ `SHOW`按钮以获取更多详细信息：

    ![](img/zipkin-traces.png)

4. 如果单击dependencies按钮并搜索，您将看到服务和它们之间的流量：

    ![](img/zipkin-dependencies.gif)

## 下一个任务

在继续下一个任务之前，请确保停止所有正在运行的进程并关闭 VS Code 中的所有终端窗口。

跳到[作业 3](../Assignment03/README.md) 。
