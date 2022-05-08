---
layout: post
title:  "SqlServer com AspNetCore no Linux"
date:   05/08/2022 14:15:00 -0300
tags: dotnet aspnet linux sqlserver
---

### Como utilizar SqlServer no linux com dotnet-core e aspnet

Siga os procedimentos

~~~shell
dotnet new mvc --auth Individual -o AnotherShop
~~~

Entre na pasta

~~~shell
cd AnotherShop/
~~~

Projeto Criado, como não usaremos o Sqlite delete o app.db e as migrations

~~~shell
rm -r app.db Data/Migrations/
~~~

Agora precisamos mudar no arquivo Startup.cs

Localize 'UseSqlite' e substitua por 'UseSqlServer', ou simplesmente execute

~~~shell
sed -i 's,UseSqlite,UseSqlServer,g' Startup.cs
~~~

Agora precisaremos criar o usuario e o banco da dados no SqlServer, para isso, connecte no localhost com SA

~~~shell
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -p
~~~

Cria o banco de dados

~~~shell
CREATE DATABASE AnotherShopDB;
GO
~~~

Cria o login e o usuario

~~~shell
CREATE LOGIN another WITH PASSWORD = 'P@ssw0rd';
GO
~~~

Crie o usuario e de a permissão db_owner para ele

~~~shell
USE AnotherShopDB;
GO
CREATE USER another FOR LOGIN another;
EXEC sp_addrolemember 'db_owner', 'another';
GO
~~~

No arquivo appsetting.json, apague o objeto ConnectionStrings, devendo ficar assim

~~~shell
{
    "Logging": {
        "LogLevel": {
        "Default": "Warning"
        }
    },
    "AllowedHosts": "*"
}
~~~

Substitua o contrutor da classe Startup.cs

~~~csharp
public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder();
    if (env.IsDevelopment())
        builder.AddUserSecrets<Startup>();
    builder.AddEnvironmentVariables();
    Configuration = builder.Build();
}
~~~

Adicione a ferramenta global user-secrets

~~~shell
dotnet tool install --global dotnet-user-secrets
~~~

Adicione a connection string

~~~shell
dotnet user-secrets set ConnectionStrings:DefaultConnection 'Server=localhost;Database=AnotherShopDB;User=another;Password=P@ssw0rd;'
~~~

Crie a migration que apagamos agora pouco

~~~shell
dotnet ef migrations add CreateIdentitySchema -o ./Data/Migrations/
dotnet ef database update
~~~

E finalmente podemos iniciar a aplicação

~~~shell
dotnet run
~~~

Seu ambiente de desenvolvimento está pronto.

## Em Produção

Em ambiende de Produção, você não deve colocar a connection string com user-secrets ou em um arquivo, para isso devemos usar as variaveis de ambiente, no linux use para registrar na variavel de ambiente

~~~shell
export SQLCONNSTR_DefaultConnection='Server=localhost;Database=AnotherShopDB;User=another;Password=P@ssw0rd;'
~~~

Ou então

~~~shell
export ConnectionStrings__DefaultConnection='Server=localhost;Database=AnotherShopDB;UserId=another;Password=P@ssw0rd;'
~~~

Note que usamos 2 _ (underline) para a substituição do : (dois pontos), visto que no linux não é possivel colocar : como parte do nome da variavel de ambiente.

## Referencias

[How to correctly store connection strings](https://stackoverflow.com/questions/44931613/how-to-correctly-store-connection-strings-in-environment-variables-for-retrieval)