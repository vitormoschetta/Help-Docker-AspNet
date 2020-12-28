# Metodo 01 (Produção Local) - Montar imagem docker apenas com dll

### Considerando a seguinte estrutura de arquivos:
```
Dockerfile
Projeto/
   arquivos do projeto
```


### O arquivo Dockerfile:
```
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
COPY Projeto/bin/Release/netcoreapp3.1/publish/ App/
WORKDIR /App
ENTRYPOINT ["dotnet", "Projeto.dll"]
```

### Publicar as dll fora da imagem primeiro:
Obs: O comando abaixo deve ser executado dentro da pasta 'Projeto':
```
dotnet publish -c Release
```

### Criar Imagem:
```
docker build -t netcore-prod-local .
```
Obs: **netcore-prod-local** é o nome dado a imagem criada.

### Publicar um Container local executando a aplicação:
```
docker run -d -p 8080:80 --name container-netcore netcore-prod-local
```
Obs: **container-netcore** é o nome dado ao container criado.  
     **netcore-prod-local** é a referência da imagem utilizada para criar o container.


Você possivelmente conseguirá acessar a aplicação na seguinte url:  
http://localhost:8080  

Também é possível acessar pelo IP do seu computador:  

IPV4:8080




---




# Metodo 02 (Produção Azure): 

Considerando a mesma estrutura do exemplo anterior.

### O arquivo Dockerfile:
É o mesmo do exemplo anterior

### Publicar as dll:
```
dotnet publish -c Release
```

### Criar Imagem:
```
docker build -t netcore-prod-azure .
```

### Criar Registro de Contêiner do Azure:  
[Tutorial](https://docs.microsoft.com/pt-br/learn/modules/deploy-run-container-app-service/3-exercise-build-images?pivots=csharp)

Obs: Necessário ter uma assinatura no Azure. 

### Login no Azure (Necessário ter o Azure CLI instalado):
```
az login
```
Obs: Uma página web será aberta pra efetuar o login

### Enviar imagem ao Registro de Contêiner do Azure:  
```
az acr build --registry container-netcore --image netcore-prod-azure .
```
Obs: **container-netcore** é o nome do registro de container criado pelo usuário que possui assinatura no Azure.  
**netcore-prod-azure** é o nome da imagem que geramos anteriormente.  

### Criar Aplicativo baseado em Container do Azure
Seguir tutorial:
[Tutorial](https://docs.microsoft.com/pt-br/learn/modules/deploy-run-container-app-service/5-exercise-deploy-web-app?pivots=csharp)




---




# Metodo 03 (Produção Heroku): 

Considerando a mesma estrutura do exemplo anterior.

### O arquivo Dockerfile:
Só difere a última linha do exemplo anterior:
```
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
COPY Projeto/bin/Release/netcoreapp3.1/publish/ App/
WORKDIR /App
CMD ASPNETCORE_URLS=http://*:$PORT dotnet Projeto.dll
```

### Publicar as dll:
```
dotnet publish -c Release
```

### Criar Imagem:
```
docker build -t netcore-prod-heroku .
```

### Login no heroku (Necessário ter o Heroku CLI instalado):
```
heroku login
```
Obs: Uma página web será aberta pra efetuar o login

### Login no Container do Heroku:
```
heroku container:login
```

### Enviar imagem ao Heroku Container:
```
heroku container:push web -a vithor-netcore
```
Obs: **vithor-netcore** é o nome do app hospedado no Heroku

### Liberar execução do Container:
```
heroku container:release web -a vithor-netcore
```




---




# Metodo 04 (Desenvolvimento Local) - Montar imagem docker com o SDK NET Core:

### Considerando a mesma estrutura de arquivos dos exemplos anteriores.

### O arquivo Dockerfile:
```
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build-env

COPY Projeto/Projeto.csproj App/
# ou:
# COPY Projeto/*.csproj App/

WORKDIR /App
RUN dotnet restore

COPY Projeto/. .
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
WORKDIR /App
COPY --from=build-env /App/out ./

ENTRYPOINT ["dotnet", "Projeto.dll"]
```

Obs: Pode acontecer de no seu projeto existirem nomes conflitantes, e você receber o seguinte erro:  
_Could not copy the 
file "/App/obj/Release/netcoreapp3.1/Projeto" to the destination file "bin/Release/netcoreapp3.1/Projeto", because the destination is a folder instead of a file. To copy the source file into a folder, consider using the DestinationFolder parameter instead of DestinationFiles. [/App/Projeto.csproj]_

Se isso acontecer, apenas mude o nome da pasta e do .csproj de '**Projeto**' para '**src**' por exemplo.


### Criar Imagem:
```
docker build -t netcore-dev-local .
```
Obs: **netcore-dev-local** é o nome dado a imagem criada.

### Publicar um Container local executando a aplicação:
```
docker run -d -p 8080:80 --name container-netcore netcore-dev-local
```
Obs: **container-netcore** é o nome dado ao container criado.  
     **netcore-dev-local** é a referência da imagem utilizada para criar o container.


Você possivelmente conseguirá acessar a aplicação na seguinte url:  
http://localhost:8080

Obs: Veja que desta vez não precisamos produzir as **dll** antes, pois o SDK fará esse trabalho conforme especificamos no Dockerfile.   
Obs2: Isso certamente deixará a execução do container mais lento.




---




# Metodo 05 (Desenvolvimento Local COM CAMADAS - DDD Backend):

### Considerando a seguinte estrutura de arquivos:
```
Dockerfile
Domain
Infra
Api
Tests
```

### O arquivo Dockerfile:
```
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build

COPY *.sln App/
COPY Domain/*.csproj App/Domain/
COPY Infra/*.csproj App/Infra/
COPY Tests/*.csproj App/Tests/
COPY Api/*.csproj App/Api/

WORKDIR /App
RUN dotnet restore

COPY Api/. ./Api/
COPY Domain/. ./Domain/
COPY Infra/. ./Infra/
COPY Tests/. ./Tests/

WORKDIR /App/Api
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS runtime
WORKDIR /App
COPY --from=build /App/Api/out ./

ENTRYPOINT ["dotnet", "api.dll"]
```

### Criar Imagem:
```
docker build -t api-dev-local .
```

### Publicar um Container local executando a aplicação:
```
docker run -d -p 8080:80 --name container-api api-dev-local
```

Obs: Estamos levando em consideração que não há acesso à nenhuma Base de Dados. Apenas dados em Memória.




---





# Metodo 06 (Produção COM CAMADAS - DDD Backend):

### Considerando a seguinte estrutura de arquivos:
```
Dockerfile
Domain
Infra
Api
Tests
```

### O arquivo Dockerfile:
```
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
COPY Api/bin/Release/netcoreapp3.1/publish/ App/src/
WORKDIR /App/src/
ENTRYPOINT ["dotnet", "api.dll"]
```

### Publicar dll
Dentro da pasta Api executar:
```
dotnet publish -c Release
```
Obs: Todas as _dll_ vão para a pasta da **API**. Por isso só precisamos pegar os arquivos da Api.

### Criar Imagem:
```
docker build -t api-prod .
```

### Publicar um Container local executando a aplicação:
```
docker run -d -p 8080:80 --name container-api-prod api-prod
```




---


