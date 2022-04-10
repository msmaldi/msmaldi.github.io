---
layout: post
title:  "Teste de Integração com dotnet core no linux."
date:   10/04/2022 14:15:00 -0300
mdate:  10/04/2022 14:15:00 -0300
tags: dotnet aspnet linux
---

### Neste exemplo, criaremos um website, executamos o servidor e testaremos com o browser Chromium.

Vamos a prática:

~~~shell
mkdir Website
cd Website
mkdir src test
dotnet new razor -o src/Website
dotnet new xunit -o test/Website.Test
dotnet add test/Website.Test reference src/Website
dotnet new sln
dotnet sln add **/**/*.csproj
~~~

Já podemos testar o codigo:

~~~shell
dotnet test
~~~

O teste já funciona normalmente.

Adicione o projeto Coypu:

~~~shell
dotnet add test/Website.Test/ package Coypu --version 3.1.1
dotnet add test/Website.Test/ package coverlet.msbuild --version 2.7.0
~~~

Agora criaremos uma pasta para executar a aplicação no teste, então crie uma pasta chama Application no projeto de teste com:

~~~shell
mkdir test/Website.Test/Application
~~~

Adicione uma classe ChromeSeleniumWebDriver.cs nesta pasta, que deve ficar assim:

~~~csharp
using Coypu.Drivers.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.Remote;
 
namespace Website.Test.Application
{
    public class ChromeSeleniumWebDriver : SeleniumWebDriver
    {
        public ChromeSeleniumWebDriver(bool headless = true)
            : base(CustomProfileDriver(headless), Coypu.Drivers.Browser.Chrome)
        {
        }
 
        private static RemoteWebDriver CustomProfileDriver(bool headless = true)
        {
            var options = new ChromeOptions();
            if (headless)
                options.AddArguments("--headless", "--disable-gpu");
 
            var service = ChromeDriverService.CreateDefaultService();
            service.HideCommandPromptWindow = true;
            service.EnableVerboseLogging = false;
            service.SuppressInitialDiagnosticInformation = true;
             
            return new ChromeDriver(service, options);
        }
    }
}
~~~

Adiciona uma classe ApplicationAmbient.cs tambem nesta, esta classe deve ficar assim:

~~~csharp
using System;
using Coypu;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;
using Xunit;
 
namespace Website.Test.Application
{
    [CollectionDefinition("ApplicationAmbient")]
    public class ApplicationAmbient : ICollectionFixture<AmbientFixture>
    {
    }
 
    public class AmbientFixture : IDisposable
    {
        const string appHost = "https://localhost";
        const UInt16 appPort = 5002;
 
        private IHost webHost;
        public BrowserSession Browser { get; }
        public AmbientFixture()
        {
            webHost = Host.CreateDefaultBuilder()
                .ConfigureWebHostDefaults(ConfigureWebHost)
                .Build();
            webHost.RunAsync();
 
            Browser = new BrowserSession(ConfigureSession(), new ChromeSeleniumWebDriver(false));   
        }
 
        static SessionConfiguration ConfigureSession()
        {
            return new SessionConfiguration
            {
                AppHost = appHost,
                Port = appPort,
                Browser = Coypu.Drivers.Browser.Chrome,
                Timeout = TimeSpan.FromSeconds(15)
            };
        }
 
        static void ConfigureWebHost(IWebHostBuilder webHost)
        {
            webHost.UseEnvironment("Testing")
                    .UseStartup<Website.Startup>()
                    .UseUrls($"{appHost}:{appPort}")
                    .UseKestrel();
        }
 
        public void Dispose()
        {
            Browser.Dispose();
        }
    }
}
~~~

Agora crie uma pasta Pages e adicione a classe HomeTest.cs que deve ficar assim:

~~~csharp
using Coypu;
using Website.Test.Application;
using Xunit;
 
namespace Website.Test.Pages
{
    [Collection("ApplicationAmbient")]
    public class HomeTest
    {
        private BrowserSession browser;
 
        public HomeTest(AmbientFixture fixture)
        {
            browser = fixture.Browser;
        }
 
        [Fact]
        public void IndexTest()
        {
            browser.Visit("/");
            Assert.True(browser.HasContent("Welcome"));
        }        
 
        [Fact]
        public void PrivacyTest()
        {
            browser.Visit("/");
            browser.ClickLink("Privacy", Options.First);
            Assert.True(browser.HasContent("Privacy Policy"));
            Assert.True(browser.HasContent("Use this page to detail your site's privacy policy."));
        }     
    }
}
~~~

Os projetos atuais do dotnet não geram a classe Startup.cs

~~~csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.HttpsPolicy;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace Website
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddRazorPages();
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Error");
                app.UseHsts();
            }

            app.UseHttpsRedirection();
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapRazorPages();
            });
        }
    }
}
~~~

E altere o Program.cs para:

~~~csharp
var builder = Host.CreateDefaultBuilder(args)
                  .ConfigureWebHostDefaults(w => w.UseStartup<Website.Startup>() );

var app = builder.Build();

app.Run();
~~~

Adicione os certificados para que funcione no Edge ou Chrome:

[Confiar no certificado HTTPS no Linux usando o Edge ou o Chrome](https://docs.microsoft.com/pt-br/aspnet/core/security/enforcing-ssl?view=aspnetcore-6.0&tabs=visual-studio#trust-https-certificate-on-linux-using-edge-or-chrome)

E para que tudo funcione, precisamos dos pacotes chromium-browser e chromium-chromedriver:

~~~shell
sudo apt install chromium-browser chromium-chromedriver
~~~

Agora execute o código e assista o teste:

~~~shell
dotnet test /p:CollectCoverage=true
~~~