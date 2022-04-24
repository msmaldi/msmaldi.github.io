---
layout: post
title:  "Usando Postgres no dotnet"
date:   24/04/2022 13:00:00 -0300
date:   24/04/2022 13:00:00 -0300
tags: postgres db dotnet ef
---

### Instalação e uso de postgres no dotnet

Tenha certeza que o postgres está instalados

~~~bash
sudo apt install -y postgres
~~~

Com o postgres instalado, vamos criar o usuário para o postgres

~~~bash
sudo -u postgres psql

CREATE USER msmaldi WITH LOGIN PASSWORD 'msmaldi' CREATEDB;
\q
~~~

Agora podemos criar o projeto

~~~bash
dotnet new console -o EfPostgres
~~~

Acesse a posta e instale o EF com suporte ao postgres.
Por convensão o postgres usa Snake Case no nome das tabelas.

~~~bash
cd EfPostgres
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add package EFCore.NamingConventions
~~~

Altere o Program.cs

~~~csharp
using Microsoft.EntityFrameworkCore;

using var ctx = new BlogContext();

Console.WriteLine("Deletando...");
await ctx.Database.EnsureDeletedAsync();
Console.WriteLine("Criando...");
await ctx.Database.EnsureCreatedAsync();

ctx.Blogs.Add(new("Another Blog"));
Console.WriteLine("Salvando...");
await ctx.SaveChangesAsync();

Console.WriteLine("Consultando...");
foreach (var blog in ctx.Blogs)
{
    System.Console.WriteLine($"{blog.Id} - {blog.Name}");
}

public class BlogContext : DbContext
{
    public DbSet<Blog> Blogs => Set<Blog>();

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseNpgsql(@"Host=localhost;Username=msmaldi;Password=msmaldi;Database=Blog")
                      .UseSnakeCaseNamingConvention();
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
                    .Property(b => b.Name)
                    .HasMaxLength(20);
    }        
}

public class Blog
{
    public Guid Id { get; set; }
    public string Name { get; set; }

    public Blog(string name)
    {
        Name = name;        
    }
}
~~~

Então é isso.
