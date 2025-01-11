# apinuvemcomo
Como fazer deploy de uma API na nuvem


Fazer o **deploy de uma API na nuvem (Azure)** envolve alguns passos, incluindo a criação de uma API em **C#** (ASP.NET Core) e a configuração de um pipeline de **CI/CD** usando um arquivo **YAML** no **Azure DevOps** ou **GitHub Actions**. A seguir, fornecerei um guia prático com exemplos de código.

### 1. **Criação da API em C# (ASP.NET Core)**

Primeiro, você precisa criar uma API simples com ASP.NET Core.

#### Passos para criar a API:
1. **Crie um novo projeto API**:
   ```bash
   dotnet new webapi -n MyApi
   cd MyApi
   ```

2. **Exemplo de controlador** (`Controllers/WeatherForecastController.cs`):

   ```csharp
   using Microsoft.AspNetCore.Mvc;
   using System;
   using System.Linq;

   namespace MyApi.Controllers
   {
       [Route("api/[controller]")]
       [ApiController]
       public class WeatherForecastController : ControllerBase
       {
           private static readonly string[] Summaries = new[]
           {
               "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
           };

           private readonly ILogger<WeatherForecastController> _logger;

           public WeatherForecastController(ILogger<WeatherForecastController> logger)
           {
               _logger = logger;
           }

           [HttpGet]
           public IEnumerable<WeatherForecast> Get()
           {
               return Enumerable.Range(1, 5).Select(index => new WeatherForecast
               {
                   Date = DateTime.Now.AddDays(index),
                   TemperatureC = Random.Shared.Next(-20, 55),
                   Summary = Summaries[Random.Shared.Next(Summaries.Length)]
               })
               .ToArray();
           }
       }

       public class WeatherForecast
       {
           public DateTime Date { get; set; }
           public int TemperatureC { get; set; }
           public string Summary { get; set; }
       }
   }
   ```

3. **Compile e execute localmente**:
   ```bash
   dotnet build
   dotnet run
   ```

   A API deve estar acessível em `http://localhost:5000/api/weatherforecast`.

### 2. **Publicação da API no Azure**

Agora, vamos configurar o **deploy** da sua API no Azure usando **Azure App Service**.

#### 2.1 **Configuração do Azure App Service**:

1. **Crie um App Service no portal do Azure**:
   - Vá para o [portal do Azure](https://portal.azure.com/).
   - Clique em "Criar um recurso" e selecione "App Service".
   - Preencha as informações do App Service (como nome, plano de hospedagem, etc.) e crie o recurso.

2. **Obtenha as credenciais de publicação**:
   - No portal do Azure, acesse o **App Service** criado.
   - Vá até "Configurações" > "Publicação" > "Credenciais de publicação" e gere um nome de usuário e senha.

#### 2.2 **Deploy da API para o Azure via GitHub Actions ou Azure DevOps**

Você pode usar o **GitHub Actions** ou **Azure DevOps** para configurar um pipeline de CI/CD. Aqui, vou usar um exemplo de **GitHub Actions**.

### 3. **Usando GitHub Actions para Deploy no Azure**

#### 3.1 **Criação do arquivo `yml` de pipeline**:

1. No seu repositório do GitHub, crie o seguinte diretório:
   ```
   .github/workflows
   ```

2. Crie o arquivo `azure-deploy.yml` dentro do diretório `workflows`:

   **Exemplo de pipeline `azure-deploy.yml`:**

   ```yaml
   name: Deploy to Azure Web App

   on:
     push:
       branches:
         - main # ou o nome do branch que você deseja monitorar

   jobs:
     build:
       runs-on: ubuntu-latest
       
       steps:
         - name: Checkout code
           uses: actions/checkout@v2

         - name: Set up .NET Core
           uses: actions/setup-dotnet@v1
           with:
             dotnet-version: '6.0'  # ou a versão que você está usando

         - name: Install dependencies
           run: |
             dotnet restore
             dotnet build

         - name: Publish the app
           run: |
             dotnet publish --configuration Release --output ./publish

         - name: Deploy to Azure Web App
           uses: azure/webapps-deploy@v2
           with:
             app-name: <nome-do-app-service>  # Nome do seu App Service
             publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}  # Defina o arquivo de publicação como segredo
             package: ./publish
   ```

#### 3.2 **Configuração do segredo `AZURE_WEBAPP_PUBLISH_PROFILE`**:

1. **Obtenha o perfil de publicação**:
   - No portal do Azure, vá para o seu **App Service**.
   - Vá para "Configurações" > "Publicação" > "Obter perfil de publicação".
   - Baixe o arquivo `.PublishSettings`.

2. **Adicionar o segredo no GitHub**:
   - Vá para o repositório no GitHub, clique em "Settings" > "Secrets".
   - Clique em "New repository secret" e adicione o segredo com o nome `AZURE_WEBAPP_PUBLISH_PROFILE` e cole o conteúdo do arquivo `.PublishSettings`.

#### 3.3 **Rodar o pipeline**:

Sempre que houver um `push` para o branch `main`, o GitHub Actions rodará o pipeline e fará o deploy automaticamente da API para o **App Service** no Azure.

### 4. **Testando a API no Azure**

Após o deploy ser bem-sucedido, a sua API estará disponível na URL do **App Service** no Azure, por exemplo:

```
https://<nome-do-app-service>.azurewebsites.net/api/weatherforecast
```

Esse é o básico para fazer o **deploy de uma API ASP.NET Core** no **Azure** usando **GitHub Actions** e o **App Service**.
