---
layout: post
title:  "Paginas de StatusCode no AspNet."
date:   03/04/2022 18:00:00 -0300
mdate:  03/04/2022 18:00:00 -0300
tags: dotnet aspnet linux
---

### Neste post, mostrarei como retornar paginas 404 (NotFound) e 500 (InternalServerError) no AspNet.

Então vamos criar um Projeto novo:

~~~shell
dotnet new razor -o Website
~~~

Mude a Environment para Production no arquivo Properties/launchSettings.json como abaixo, visto que esses direcionamentos deve ser feitos somente em produção, e é neles que testaremos:

~~~shell
{
    ...
    "Website": {
        "commandName": "Project",
        "launchBrowser": true,
        "applicationUrl": "https://localhost:5001;http://localhost:5000",
        "environmentVariables": {
            "ASPNETCORE_ENVIRONMENT": "Production"
        }
    }
}
~~~

Inicie o aplicativo, vejá que se tentar acessar uma URL que não existe, ele retorna uma pagina com o StatusCode 404 mas com o Body vazio.

Então altere o arquivo Error.cshtml.cs para:

~~~csharp
using System.Net;

...

[ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
public class ErrorModel : PageModel
{
    public int StatusCodeId { get; private set; }
    public HttpStatusCode HttpStatusCode { get; private set; }
    public string Message { get; private set; }
 
    public void OnGet()
    {
        PrepareStatusCode();
    }
 
    public void PrepareStatusCode()
    {
        if (Response.StatusCode == (int)HttpStatusCode.OK)
        {
            Response.StatusCode = 404;
        }
 
        StatusCodeId = Response.StatusCode;
        HttpStatusCode = (HttpStatusCode)StatusCodeId;
        Message = Messages[HttpStatusCode];
    }
 
    static Dictionary<HttpStatusCode, string> Messages =
        new Dictionary<HttpStatusCode, string>()
        {
            { HttpStatusCode.NotFound, "O recurso solicitado não existe no servidor." },
            { HttpStatusCode.InternalServerError, "Ocorreu um erro interno em nosso servidor." }
        };
}
~~~

E o arquivo Error.cshtml para:

~~~csharp
@page
@model ErrorModel
@{ ViewData["Title"] = Model.HttpStatusCode; }
<style>
    .sorry {
        font-size: 72px;
        font-family: 'Courier New', Courier, monospace;
        margin-top: 50px;
        margin-bottom: 50px;
    }
</style>
<div class="text-center">
    <h2 class="text-danger">
        @Model.StatusCodeId @Model.HttpStatusCode
    </h2>
    <p>
        @Model.Message
    </p>
    <p>
        E é tudo que sabemos.
    </p>
    <p class="sorry">=[</p>
</div>
~~~

E no Startup.cs altere o metodo configure o metodo UseStatusCodePagesWithReExecute

~~~csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseStatusCodePagesWithReExecute("/Error");
        app.UseHsts();
    }
 
    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
 
    app.UseMvc();
}
~~~

Caso não faça o uso do Startup.cs, altere no Program.cs

~~~csharp
// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseStatusCodePagesWithReExecute("/Error");
}
~~~

Assim já podemos ver as paginas de StatusCode 404 e 500 personalizadas.

Vejá com o wget os Status Code de resposta

~~~shell
wget -S --spider -q --no-check-certificate https://localhost:5001
~~~

~~~shell
  HTTP/1.1 200 OK
  Date: Fri, 30 Nov 2018 10:47:13 GMT
  Content-Type: text/html; charset=utf-8
  Server: Kestrel
~~~

~~~shell
wget -S --spider -q --no-check-certificate https://localhost:5001/NotFound
~~~

~~~shell
  HTTP/1.1 404 Not Found
  Date: Fri, 30 Nov 2018 10:59:39 GMT
  Content-Type: text/html; charset=utf-8
  Server: Kestrel
  Cache-Control: no-store,no-cache
  Pragma: no-cache
~~~

O que acontece se tentar acessar a rota /Error

~~~shell
wget -S --spider -q --no-check-certificate https://localhost:5001/Error
~~~

~~~shell
  HTTP/1.1 404 Not Found
  Date: Fri, 30 Nov 2018 11:01:13 GMT
  Content-Type: text/html; charset=utf-8
  Server: Kestrel
  Cache-Control: no-store,no-cache
  Pragma: no-cache
~~~

Note que retorna uma pagina 404, isso porque efetuamos o tratamento para que o cliente não saiba que a pagina /Error exista.

## Referencias

[Re-execute the middleware pipeline with the StatusCodePages Middleware to create custom error pages](https://andrewlock.net/re-execute-the-middleware-pipeline-with-the-statuscodepages-middleware-to-create-custom-error-pages/)


