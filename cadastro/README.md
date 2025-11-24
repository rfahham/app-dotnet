A seguir vou montar **uma arquitetura completa** usando **Docker Compose**, contendo:

* 🟦 **Container 1**: Aplicação .NET chamada **cadastro** (Minimal API + CRUD)
* 🟩 **Container 2**: **SQL Server** para armazenar Nome + Data de Nascimento
* 🔗 Conexão entre os dois containers via Docker network
* 🔧 Volume para persistência do banco
* ⚙️ Variáveis de ambiente configuradas corretamente

Vou entregar:

1. **docker-compose.yml pronto**
2. **Dockerfile da aplicação**
3. **Exemplo de Minimal API com CRUD completo**
4. **Model + DbContext usando EF Core**

---

# 🟦 1. docker-compose.yml

Crie um arquivo chamado **docker-compose.yml** na raiz:

```yaml
version: "3.9"

services:
  cadastro-api:
    build:
      context: ./cadastro
      dockerfile: Dockerfile
    container_name: cadastro-api
    depends_on:
      - sqlserver
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=sqlserver;Database=CadastroDb;User=sa;Password=Your_strong_password123;TrustServerCertificate=True;
    ports:
      - "5000:80"
    networks:
      - cadastro-net

  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: cadastro-sql
    environment:
      SA_PASSWORD: "Your_strong_password123"
      ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"
    volumes:
      - sql_data:/var/opt/mssql
    networks:
      - cadastro-net

networks:
  cadastro-net:
    driver: bridge

volumes:
  sql_data:
```

---

# 🟧 2. Dockerfile da aplicação .NET

Na pasta `cadastro/` crie um **Dockerfile**:

```dockerfile
# Etapa 1: Build
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o /publish

# Etapa 2: Runtime
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS runtime
WORKDIR /app
COPY --from=build /publish .

EXPOSE 80

ENTRYPOINT ["dotnet", "cadastro.dll"]
```

---

# 🟩 3. Estrutura do projeto .NET

```
cadastro/
 ├── Program.cs
 ├── Dockerfile
 ├── cadastro.csproj
 └── Data/
      ├── CadastroContext.cs
 └── Models/
      ├── Pessoa.cs
```

---

# 📌 4. Classe Pessoa (Nome + DataNascimento)

**Models/Pessoa.cs**

```csharp
namespace cadastro.Models
{
    public class Pessoa
    {
        public int Id { get; set; }
        public string Nome { get; set; } = "";
        public DateOnly DataNascimento { get; set; }
    }
}
```

---

# 📌 5. EF Core DbContext

**Data/CadastroContext.cs**

```csharp
using cadastro.Models;
using Microsoft.EntityFrameworkCore;

namespace cadastro.Data
{
    public class CadastroContext : DbContext
    {
        public CadastroContext(DbContextOptions<CadastroContext> options)
            : base(options) { }

        public DbSet<Pessoa> Pessoas => Set<Pessoa>();
    }
}
```

---

# 📌 6. Program.cs com CRUD completo

```csharp
using cadastro.Data;
using cadastro.Models;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// configurar EF Core
builder.Services.AddDbContext<CadastroContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// criar banco na inicialização
using (var scope = app.Services.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<CadastroContext>();
    db.Database.Migrate();
}

// rotas CRUD
app.MapPost("/pessoas", async (Pessoa pessoa, CadastroContext db) =>
{
    db.Pessoas.Add(pessoa);
    await db.SaveChangesAsync();
    return Results.Created($"/pessoas/{pessoa.Id}", pessoa);
});

app.MapGet("/pessoas", async (CadastroContext db) =>
    await db.Pessoas.ToListAsync());

app.MapGet("/pessoas/{id}", async (int id, CadastroContext db) =>
{
    var pessoa = await db.Pessoas.FindAsync(id);
    return pessoa is not null ? Results.Ok(pessoa) : Results.NotFound();
});

app.MapPut("/pessoas/{id}", async (int id, Pessoa input, CadastroContext db) =>
{
    var pessoa = await db.Pessoas.FindAsync(id);
    if (pessoa is null) return Results.NotFound();

    pessoa.Nome = input.Nome;
    pessoa.DataNascimento = input.DataNascimento;

    await db.SaveChangesAsync();
    return Results.NoContent();
});

app.MapDelete("/pessoas/{id}", async (int id, CadastroContext db) =>
{
    var pessoa = await db.Pessoas.FindAsync(id);
    if (pessoa is null) return Results.NotFound();

    db.Pessoas.Remove(pessoa);
    await db.SaveChangesAsync();
    return Results.Ok(pessoa);
});

app.Run();
```

---

# 🏁 7. Rodar tudo

Na pasta raiz:

```bash
docker-compose up -d --build
```

API disponível em:

```
http://localhost:5000/pessoas
```

Exemplo para criar pessoa:

```json
POST /pessoas
{
  "nome": "João da Silva",
  "dataNascimento": "1990-05-22"
}
```

---

# 🎉 Pronto!

Você tem:

✔ Arquitetura limpa com API + banco SQL Server
✔ CRUD completo
✔ Dockerfile
✔ Docker Compose
✔ EF Core com Migrations automáticas

Podemos adicionar:

🔹 Swagger
🔹 Healthcheck
🔹 Seed inicial de dados
🔹 Testes automáticos (K6 ou Postman)
🔹 Pipeline CI/CD


