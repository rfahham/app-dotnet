# Como criar um **pipeline CI/CD** simples para a sua aplicação **.NET Minimal API** com **Docker**, usando **GitHub Actions** como exemplo. Ele vai:

1. Buildar a aplicação
2. Executar testes (caso existam)
3. Criar a imagem Docker
4. Publicar a imagem no Docker Hub (ou outro registry)

---

# ✅ Estrutura do GitHub Actions

Crie a pasta `.github/workflows/` na raiz do projeto e adicione um arquivo, por exemplo `ci-cd.yml`:

```yaml
name: CI/CD .NET Docker

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    env:
      DOCKER_IMAGE: seuusuario/meuapp-dotnet

    steps:
      # 1️⃣ Checkout do código
      - name: Checkout repository
        uses: actions/checkout@v3

      # 2️⃣ Setup .NET
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '9.0.x'

      # 3️⃣ Restore dependências
      - name: Restore dependencies
        run: dotnet restore

      # 4️⃣ Build
      - name: Build
        run: dotnet build --configuration Release --no-restore

      # 5️⃣ Testes (opcional)
      - name: Run tests
        run: dotnet test --no-build --verbosity normal

      # 6️⃣ Publish
      - name: Publish
        run: dotnet publish -c Release -o ./publish

      # 7️⃣ Login Docker Hub
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      # 8️⃣ Build Docker image
      - name: Build Docker image
        run: docker build -t $DOCKER_IMAGE:latest .

      # 9️⃣ Push Docker image
      - name: Push Docker image
        run: docker push $DOCKER_IMAGE:latest
```

---

# 🔹 Passo a passo

1. **Adicionar secrets no GitHub**

   * `DOCKER_USERNAME` → seu usuário Docker Hub
   * `DOCKER_PASSWORD` → sua senha ou token

2. **Estrutura do projeto** deve conter:

   * `Program.cs`
   * `Dockerfile`
   * `.csproj`

3. **Branch principal** (`main`) será usada como gatilho.

4. **Saída esperada:**

   * Build .NET bem-sucedido
   * Testes executados
   * Imagem Docker criada e publicada no Docker Hub

---

# 🔹 Observações

* Você pode adicionar **deploy automático** em um servidor (AWS ECS, Azure App Service, DigitalOcean, etc.) usando etapas adicionais.
* Para **testes locais de Docker**, antes de push:

```bash
docker build -t meuapp-dotnet .
docker run -p 5000:80 meuapp-dotnet
```

* Opção **pipeline completo com HealthCheck**, 404 customizado e variáveis de ambiente,criar uma versão pronta para deploy **em produção**.
