# Metodo 01 (Produção) - Montar imagem docker apenas com dll

#### Considerando a seguinte estrutura de arquivos:
```
Dockerfile
Projeto/
   arquivos do projeto
```


#### O arquivo Dockerfile:
```
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1
COPY Projeto/bin/Release/netcoreapp3.1/publish/ App/
WORKDIR /App
ENTRYPOINT ["dotnet", "Projeto.dll"]
```

#### Publicar as dll fora da imagem primeiro:
Obs: O comando abaixo deve ser executado dentro da pasta 'Projeto':
```
dotnet publish -c Release
```

#### Criar Imagem:
```
docker build -t netcore-dll .
```
Obs: **netcore-dll** é o nome dado a imagem criada.

#### Publicar um Container local executando a aplicação:
```
docker run -d -p 8080:80 --name container-netcore netcore-dll
```
Obs: **container-netcore** é o nome dado ao container criado.  
     **netcore-dll** é a referência da imagem utilizada para criar o container.


Você possivelmente conseguirá acessar a aplicação na seguinte url:
http://localhost:8080
