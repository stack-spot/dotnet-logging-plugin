## **Visão Geral**
O plugin **`dotnet-logging-app-plugin`** adiciona em uma Stack a capacidade de padronizar a escrita de logs, reduzindo a verbosidade de códigos e proporcionando celeridade e observações mais precisas.

## **Uso**

### **Pré-requisitos**
Para utilizar este plugin é preciso ter instalado na sua máquina os itens abaixo:  
- Uma Stack dotNet criada pelo [**STK CLI**](https://stackspot.com/);  
- .NET 5 ou 6 
- O template **`dotnet-api-template`** ou o **`dotnet-worker-template`** já aplicados. 

## **Configuração**
### **Inputs configurados automaticamente**  

O input abaixo é usado para configurar o plugin:  

| **Campo** | **Valor** | **Descrição** |
| :--- | :--- | :--- |
| Log Level| Padrão: "INFO" | Level de log que tem os valores: DEBUG, INFO, WARN, ERROR, FATAL. |

- As variáveis serão configuradas no arquivo **`appsettings.json`**. Confira o exemplo abaixo:  

```json
{
  "AppName": "MyAppName",  
  "LogOptions": {
    "LogLevel": "INFO"
  }
}
```
- Quando o plugin for executado na máquina local, as variáveis de ambientes serão configuradas no arquivo **`launchSettings.json`**. Confira o exemplo abaixo:  

```json
{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "profiles": {
    "Sample.WebApi": {
      "commandName": "Project",
      "launchBrowser": true,
      "launchUrl": "sample",
      "environmentVariables": {
        "APP_NAME": "MyAppName",
        "LOG_LEVEL": "INFO",
        "ASPNETCORE_ENVIRONMENT": "Development"
      },
      "applicationUrl": "http://localhost:5000"
    }
  }
}
```

- As configurações abaixo serão feitas no **`IServiceCollection`**, através do `services.AddLogger()`, no `Startup` da aplicação ou `Program`.

**Utilizando váriavel de ambiente**:  

```csharp
services.AddLogger();
```

**Utilizando o `appsettings.json`**: 

```csharp
services.AddLogger(Configuration);
```

#### **Instalação de pacotes adicionais**  
Caso precise de um log completo com informações de contexto, é possível instalar os pacotes adicionais e utilizar a configuração abaixo:

**Exemplo OpenTracing**:

```csharp
services.AddLogger()
        .WithOpenTracing()
        .WithCorrelation();
```

**Exemplo XRay**:

```csharp
services.AddLogger()
        .WithXRayTraceId()
        .WithCorrelation();
```

#### **Exemplos de utilização do plugin**  
Os métodos do `ILogger<>` foram estendidos, transformando o output. Além disso, duas novas sobrecargas estão sendo providas para suportar **TAGs** e **log de Objetos** no campo **Data**. Confira o exemplo abaixo:  

```csharp
[ApiController]
[Route("[controller]")]
public class SampleController : ControllerBase
{
    private readonly ILogger<SampleController> _logger;

    public SampleController(ILogger<SampleController> logger)
    {
        _logger = logger;
    }

    [HttpGet()]
    public async Task<IActionResult> Get()
    {
        var someEntity = new SampleEntity();
        _logger.LogDebug("My DEBUG Log Message", someEntity, "Tag01", "Tag02");
        return Ok();
    }
}
```
- **Debug**  

```csharp
_logger.LogDebug("My DEBUG Log Message");
_logger.LogDebug("My DEBUG Log Message", "Tag01", "Tag02");
_logger.LogDebug("My DEBUG Log Message", someEntity, "Tag01", "Tag02");
```

- **Info**  

```csharp
_logger.LogInformation("My INFO Log Message");
_logger.LogInformation("My INFO Log Message", "Tag01", "Tag02");
_logger.LogInformation("My INFO Log Message", someEntity, "Tag01", "Tag02");
```

- **Warning**

```csharp
_logger.LogWarning("My WARNING Log Message");
_logger.LogWarning("My WARNING Log Message", "Tag01", "Tag02");
_logger.LogWarning("My WARNING Log Message", someEntity, "Tag01", "Tag02");
```

- **Error**  

```csharp
_logger.LogError("My ERROR Log Message");
_logger.LogError("My ERROR Log Message", "Tag01", "Tag02");
_logger.LogError("My ERROR Log Message", someEntity, "Tag01", "Tag02");
```

- **Fatal**  

```csharp
_logger.LogFatal("My ERROR Log Message");
_logger.LogFatal("My ERROR Log Message", "Tag01", "Tag02");
_logger.LogFatal("My ERROR Log Message", someEntity, "Tag01", "Tag02");
```

#### Exemplo de Output completo

Confira abaixo o output completo preenchido com as informações de DotNET:  

```json
{
    "timeStamp": "2021-04-06T14:50:33.6610795Z",
    "appName": "MyAppName",
    "message": "An unhandled exception has occurred while executing the request.",
    "logger": "Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware",
    "level": "ERROR",
    "tags": ["Tag01", "Tag02"],
    "data": {
        "field1": "Test01",
        "field2": "Test02"
    },
    "exception": {
        "name": "DivideByZeroException",
        "message": "Attempted to divide by zero.",
        "stackTrace": "   at Sample.WebApi.Controllers.SampleController.Get() in ..."
    },
    "context": {
        "spanId": "1af157b8bee48886",
        "traceId": "1af157b8bee48886",
        "correlationId": "614bc03a1eab685315a897fe1405a935"
    }
}
```