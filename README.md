Aqui está a **documentação revisada, corrigida, organizada e padronizada**, mantendo o mesmo conteúdo, mas com estrutura mais clara, sem erros e com melhor formatação.

---

# 🚀 app-dotnet

Criando uma aplicação utilizando **.NET** com **Minimal API**, **Healthcheck** e execução básica.

---

# ✅ 1. Instalar o .NET SDK

## 🟦 Windows

Baixe e instale o SDK no site oficial:

➡️ [https://dotnet.microsoft.com/download](https://dotnet.microsoft.com/download)

Depois de instalar, reinicie o terminal e teste:

```bash
dotnet --version
```

---

## 🟩 Linux (Ubuntu/Debian)

```bash
sudo apt update
sudo apt install dotnet-sdk-9.0
```

Teste:

```bash
dotnet --version
```

---

## 🟧 macOS (Intel ou M1/M2/M3)

Via Homebrew:

```bash
brew install --cask dotnet-sdk
```

Adicione ao PATH caso necessário:

```bash
export PATH="$PATH:/usr/local/share/dotnet"
```

Teste:

```bash
dotnet --version
```

---

# ✅ 2. Criar o projeto

No terminal:

```bash
dotnet new webapi -n MeuProjeto
cd MeuProjeto
```

Esse template já suporta **Minimal APIs**.

---

# ✅ 3. Adicionar Health Checks

No `Program.cs`, adicione:

```csharp
var builder = WebApplication.CreateBuilder(args);

// registra o health check
builder.Services.AddHealthChecks();

var app = builder.Build();

// endpoint padrão de healthcheck
app.MapHealthChecks("/healthcheck");

app.Run();
```

Isso cria o endpoint:

```
GET /healthcheck
```

Retorno padrão:

* **200 OK** se a aplicação estiver funcionando
* Pode ser expandido com checks de banco, cache, serviços externos, etc.

---

# 🔍 4. Testar a aplicação

## 👉 Build

```bash
dotnet build
```

Exemplo de saída:

```
Build succeeded.
    0 Warning(s)
    0 Error(s)
```

---

## 👉 Executar

```bash
dotnet run
```

Saída típica:

```
Now listening on: http://localhost:5251
Application started. Press Ctrl+C to shut down.
Hosting environment: Development
```

---

# 🔗 5. Acessar endpoints

Abra no navegador:

### Healthcheck

```
http://localhost:5251/healthcheck
```

### WeatherForecast (template gerado)

```
http://localhost:5251/weatherforecast
```

### Swagger

```
http://localhost:5251/swagger
```

Retorno típico do healthcheck:

```
Healthy
```

---

# ⭐ Exemplo com verificações extras (opcional)

Se quiser adicionar, por exemplo, SQL Server:

```csharp
builder.Services.AddHealthChecks()
    .AddSqlServer(builder.Configuration.GetConnectionString("SqlServer"));
```

---

# 🧩 Healthcheck customizado (retornar “WORKING”)

## 1. Instalar o pacote necessário (se ainda não tiver)

```bash
dotnet add package Microsoft.AspNetCore.Diagnostics.HealthChecks
```

Ou, se for usar UI:

```bash
dotnet add package AspNetCore.HealthChecks.UI.Client
```

---

## 2. Adicionar os `using` no topo do Program.cs

```csharp
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using System.Text.Json;
```

---

## 3. Criar um healthcheck personalizado

```csharp
app.MapHealthChecks("/healthcheck", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "text/plain; charset=utf-8";
        await context.Response.WriteAsync("WORKING");
    }
});
```

👉 Retorna apenas:

```
WORKING
```

---

# 🔄 Executar novamente

```bash
dotnet build
dotnet run
```

Acesse:

```
http://localhost:5251/healthcheck
```

---

# 🎉 Resultado esperado

```
WORKING
```

---


