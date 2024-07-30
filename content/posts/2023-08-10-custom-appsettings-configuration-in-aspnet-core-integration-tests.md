---
title: "Custom AppSettings Configuration in ASP.NET Core Tests"
date: 2023-08-10
dateUpdated: Last Modified
permalink: /posts/custom-appsettings-configuration-in-aspnet-core-integration-tests/
tags:
  - ASP.NET
  - Testing
layout: layouts/post.njk
---

**Problem:** The ASP.NET integration tests use the web application project `appsettings.json` configuration.
When you start the web application from the integration tests using the TestServer (from the `Microsoft.AspNetCore.Mvc.Testing` package), the web application uses the `appsettings.json` configuration files that are placed in the web application root directory. Normally this doesn't suit us well since we need to have different configs for our integration tests for things like connection strings, storage URLs, etc.

**Solution:** Configure the ASP.NET integration tests to use the test project specific `appsettings.json` configuration.

![](/img/2023-08-10-using-custom-appsettings-configuration-in-aspnet-core-integration-tests-with-testserver/testserver-appsettings-diagram.png)

To do this, create `appsettings.json` and `appsettings.Development.json` configuration files in the integration tests project.

Then, set the properties `Build Action = Content` and `Copy to output directory = Copy if newer` to both files.

Finally, configure the `WebApplicationFactory` to use the configuration files from the test project itself. Here is a sample code and a test case.

```cs
using Microsoft.AspNetCore.Mvc.Testing;
using Microsoft.Extensions.Configuration;

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
                    builder.ConfigureAppConfiguration((context, config) =>
                    {
                        // Set the base path to the integration tests build output directory
                        // where the integration tests' config files will be copied into.
                        config.SetBasePath(Directory.GetCurrentDirectory());
                        config.AddJsonFile("appsettings.json");
                        config.AddJsonFile("appsettings.Development.json");
                    });
                });
        }

        [Fact]
        public async Task Test1()
        {
            var httpClient = _webAppFactory.CreateClient();

            var result = await httpClient.GetStringAsync("/");

            // Here the endpoint returns configuration specific result for testing purposes.
            Assert.Equal("ProjectKey is: 'integrationtests_local_development'", result);
        }
    }
}
```
