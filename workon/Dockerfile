# ==============================
# Etapa 1: Build
# ==============================
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /app

# Copia csproj e restaura dependências
COPY *.csproj ./
RUN dotnet restore

# Copia todo o restante do código e build
COPY . ./
RUN dotnet publish -c Release -o /app/publish

# ==============================
# Etapa 2: Runtime
# ==============================
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .

# Porta que o container vai expor
EXPOSE 80
EXPOSE 443

# Variável de ambiente para URL
ENV ASPNETCORE_URLS=http://+:80

# Comando para rodar a aplicação
ENTRYPOINT ["dotnet", "workon.dll"]


# Build da imagem
#docker build -t meuapp-dotnet .

# Rodar o container
#docker run -d -p 5000:80 --name meuapp meuapp-dotnet

# O app vai ficar acessível em http://localhost:5000/healthcheck

# O healthcheck retornará WORKING (ou o que você configurou)