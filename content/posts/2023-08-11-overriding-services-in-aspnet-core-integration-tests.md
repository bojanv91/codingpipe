---
title: "Overriding services ASP.NET Core integration tests using ConfigureTestServices"
date: 2023-08-11
dateUpdated: Last Modified
permalink: /posts/overriding-services-in-aspnet-core-integration-tests/
tags:
  - ASP.NET Core
  - Testing
layout: layouts/post.njk
---

Let's say you have the following ASP.NET Core project. 

```cs
namespace PlaygroundApi
{
    public class Program
    {
        public static void Main(string[] args)
        {
            var builder = WebApplication.CreateBuilder(args);
            builder.Services.AddScoped<IPaymentService>(x => new StripePaymentService());

            var app = builder.Build();
            app.MapGet("/", (IConfiguration config) =>
            {
                var projectKey = config.GetValue<string>("ProjectKey");
                return $"ProjectKey is: '{projectKey}'";
            });
            app.Run();
        }
    }
}
```

It has a `IPaymentService` scoped service injected with a `StripePaymentService` implementation. 

For the needs of integration testing we want to mock the payment service, and not use the real `StripePaymentService` implementation. To do that, we can use the `ConfigureTestServices` method extension in the TestServer to override the `IPaymentService` implementation with a `MockPaymentService`.

```cs
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.AspNetCore.TestHost;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace PlaygroundApi.Tests
{
    public class UnitTest1
    {
        private readonly WebApplicationFactory<PlaygroundApi.Program> _webAppFactory;

        public UnitTest1()
        {
            _webAppFactory = new WebApplicationFactory<PlaygroundApi.Program>()
                .WithWebHostBuilder(builder =>
                {
                    builder.ConfigureTestServices(services =>
                    {
                        services.AddScoped<IPaymentService>(x => new MockPaymentService());
                    });
                });
        }

        [Fact]
        public void Test_payment()
        {
            using var scope = _webAppFactory.Services.CreateScope();
            var paymentService = scope.ServiceProvider.GetRequiredService<IPaymentService>();

            // Test proof that the payment service type now is the mock.
            Assert.Equal(typeof(MockPaymentService), paymentService.GetType());
        }
    }
}
```
