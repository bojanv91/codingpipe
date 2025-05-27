---
title: "Building .NET Core Windows Services"
date: 2023-08-16
dateUpdated: Last Modified
permalink: /posts/building-windows-service-applications-in-net-core/
tags:
  - ASP.NET Core 
  - C#
layout: layouts/post.njk
---

Running your console app as a Windows Service is a great way to execute long running or time-scheduled operations in the background, outside of your web endpoints handlers.

In this post, we'll create a console application that also can be started as a Windows Service. Then, we'll add logging with *Serilog* to ensure log files will be created in the correct directories.

## Step 1 - Create new "Worker Service" project
**First**, create a new project in Visual Studio choosing the "Worker Service" template. I'll name my project `PlaygroundWorkerService`. This template creates the following folders and files structure from which we can infer how to add new functionalities:

```
appsettings.json
appsettings.Development.json
Program.cs
Worker.cs
```

`Program.cs` holds the project initialization code. 
`Worker.cs` is a sample `HostedService` implementation.

If you Ctrl+F5 start this project it will properly run as a console application in the terminal.

## Step 2 - Setup the Windows Services hosting extension
**Next**, install the *Microsoft.Extensions.Hosting.WindowsServices* NuGet package to the project:

```
dotnet add package Microsoft.Extensions.Hosting.WindowsServices
```

**Next**, call the `.UseWindowsService()` extension method in `Program.cs`, just before `.Build()`. 
**And then,** set the current directory to the actual base directory of your app - a fix to the base app path for Windows Service apps.

```diff-csharp
namespace PlaygroundWorkerService
{
    public class Program
    {
        public static void Main(string[] args)
        {
+	        Directory.SetCurrentDirectory(AppDomain.CurrentDomain.BaseDirectory);

            IHost host = Host.CreateDefaultBuilder(args)
                .ConfigureServices(services =>
                {
                    services.AddHostedService<Worker>();
                })
+                .UseWindowsService()
                .Build();

            host.Run();
        }
    }
}
```

**NOTE:** When the app runs as a service, `.UseWindowsService()` sets the [ContentRootPath](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.ihostenvironment.contentrootpath#microsoft-extensions-hosting-ihostenvironment-contentrootpath) to [AppContext.BaseDirectory](https://learn.microsoft.com/en-us/dotnet/api/system.appcontext.basedirectory#system-appcontext-basedirectory) which for a Windows Service defaults to `C:\Windows\system32`. This means that any log files (or any other files) that you would expect to find in your local app folder, will appear in the system32 folder. The fix for this side effect is the following line of code which we already added above:

```cs
Directory.SetCurrentDirectory(AppDomain.CurrentDomain.BaseDirectory);
```

**Now**, if you Ctrl+F5 start this project it will still properly run as a console application in the terminal, but also can be started as a Windows Service and run in the background. 

## Step 3 - Publish and host the console app as a Windows Service
Next, publish your console app to a publish output directory:

```
dotnet publish -r win-x64 --no-self-contained -c Release --output "D:\win-services\PlaygroundWorkerService"
```

**Next**, open your terminal as an administrator, navigate to your publish output directory (mine is: `D:\win-services\PlaygroundWorkerService`), and run the following command to host your console app as a Windows Service:

```
sc create "PlaygroundWorkerService" binpath="D:\win-services\PlaygroundWorkerService\PlaygroundWorkerService.exe" start="auto"
```

The terminal shows `[SC] CreateService SUCCESS` to inform you that the Windows Service has been successfully created with autostart mode enabled (meaning - when the OS restarts, the service will start automatically on startup).

**Next**, start your service:

```
sc start "PlaygroundWorkerService"
```

**Finally**, check if your service is running:

```
sc query "PlaygroundWorkerService"
```

The last command will return the following status if the service is successfully running:

```
SERVICE_NAME: PlaygroundWorkerService
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

## Additionally - Stopping and deleting a Windows Service

If you need to stop or delete your service, you can run the following commands:

```
sc stop "PlaygroundWorkerService"

sc delete "PlaygroundWorkerService"
```

## Further reading

- [Adding Serilog logger to a .NET Core Windows Service app](/2023-08-16-creating-and-deploying-a-windows-service-app-in-dotnet)
