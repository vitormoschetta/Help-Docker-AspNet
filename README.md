# Metodo 01 (Produção) - Montar imagem docker apenas com dll

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
docker build -t prod-netcore .
```
Obs: **prod-netcore** é o nome dado a imagem criada.

### Publicar um Container local executando a aplicação:
```
docker run -d -p 8080:80 --name container-netcore prod-netcore
```
Obs: **container-netcore** é o nome dado ao container criado.  
     **prod-netcore** é a referência da imagem utilizada para criar o container.


Você possivelmente conseguirá acessar a aplicação na seguinte url:
http://localhost:8080


---

# Metodo 02 (Desenvolvimento) - Montar imagem docker com o SDK NET Core:

### Considerando a seguinte estrutura de arquivos:
```
Dockerfile
Projeto/
   arquivos do projeto
```

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
docker build -t dev-netcore .
```
Obs: **dev-netcore** é o nome dado a imagem criada.

### Publicar um Container local executando a aplicação:
```
docker run -d -p 8080:80 --name container-netcore dev-netcore
```
Obs: **container-netcore** é o nome dado ao container criado.  
     **dev-netcore** é a referência da imagem utilizada para criar o container.


Você possivelmente conseguirá acessar a aplicação na seguinte url:
http://localhost:8080

Obs: Veja que desta vez não precisamos produzir as **dll** antes, pois o SDK fará esse trabalho conforme especificamos no Dockerfile.   
Obs2: Isso certamente deixará a execução do container mais lento.

---





