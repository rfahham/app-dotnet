# app-dotnet

Criando uma aplicação utilizando dot.net

# ✅ 1. Instalar o .NET SDK

## 🟦 **Windows**

Baixe e instale o SDK no site oficial:

➡️ [https://dotnet.microsoft.com/download](https://dotnet.microsoft.com/download)

Depois de instalar, feche e abra o terminal novamente e teste:

```bash
dotnet --version
```

---

## 🟩 **Linux (Ubuntu/Debian)**

```bash
sudo apt update
sudo apt install dotnet-sdk-9.0
```

Teste:

```bash
dotnet --version
```

---

## 🟧 **macOS (Intel ou M1/M2/M3)**

Via Homebrew:

```bash
brew install --cask dotnet-sdk
```

Depois adicione ao PATH (se necessário):

```bash
export PATH="$PATH:/usr/local/share/dotnet"
```

Teste:

```bash
dotnet --version
```

---


## ✅ 1. Criar o projeto

No terminal:

```bash
dotnet new webapi -n MeuProjeto
cd MeuProjeto
```

Esse template já suporta Minimal APIs.

---

## ✅ 2. Adicionar Health Checks

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
GET /healthhealthcheck
```

Retorno padrão:

* **200 OK** se a aplicação estiver funcionando
* Pode ser expandido com checks de banco, cache, serviços externos, etc.

---

## 🔍 Teste

Para Buildar

Rode:

```bash
dotnet build

MSBuild version 17.8.43+f0cbb1397 for .NET
  Determining projects to restore...
  All projects are up-to-date for restore.
  workon -> /home/rfahham/projetos/app-dotnet/workon/bin/Debug/net8.0/workon.dll

Build succeeded.
    0 Warning(s)
    0 Error(s)

Time Elapsed 00:00:01.57
```

Para executar:

```bash
dotnet run

Building...
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://localhost:5251
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /home/rfahham/projetos/app-dotnet/workon

```

Acesse:

```
E teste no navegador:

http://localhost:5251/healthcheck
 → deve retornar Healthy

http://localhost:5251/weatherforecast

http://localhost:5251/swagger
```

Você verá algo assim:

```
Healthy
```

---

## ⭐ Exemplo com verificações extras (opcional)

Se quiser já incluir checks de banco, por exemplo:

```csharp
builder.Services.AddHealthChecks()
    .AddSqlServer(builder.Configuration.GetConnectionString("SqlServer"));
```

---


* Dockerfile
* Documentação Swagger
* Um projeto completo com healthcheck + banco?



### Healthcheck customizado?

# ✅ **1. Instalar o pacote necessário (caso ainda não tenha)**

No terminal, dentro do seu projeto:

```bash
dotnet add package AspNetCore.HealthChecks.UI.Client
```

Ou, caso use .NET 8/9 (Minimal API), normalmente só precisa de:

```bash
dotnet add package Microsoft.AspNetCore.Diagnostics.HealthChecks
```

---

# ✅ **2. Adicionar os `using` no topo do Program.cs**

No topo do arquivo, coloque:

```csharp
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using System.Text.Json;
```

---

# ✅ **3. Código completo para retornar “WORKING”**

Use isto no Program.cs:

```csharp
app.MapHealthChecks("/healthcheck", new HealthCheckOptions
{
    ResponseWriter = async (context, report) =>
    {
        context.Response.ContentType = "text/plain";
        await context.Response.WriteAsync("WORKING");
    }
});
```

👉 Isso retorna **apenas o texto WORKING**, sem JSON.

---

# 🔄 Agora rode novamente:

```bash
dotnet build
dotnet run
```

E abra:

```
http://localhost:5251/healthcheck
```

Você deve ver:

```
WORKING
```


