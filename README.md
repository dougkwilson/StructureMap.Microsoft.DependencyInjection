# StructureMap integration for ASP.NET Core [![Build status](https://ci.appveyor.com/api/projects/status/tpk77374afp3dk8v?svg=true)](https://ci.appveyor.com/project/khellang/structuremap-dnx)

This repository contains the source of two NuGet packages:

 - StructureMap.AspNetCore
 - StructureMap.Microsoft.DependencyInjection

These packages provide integration with ASP.NET Core and the built-in container on different levels.

## StructureMap.AspNetCore

Adds integration with the ASP.NET Core hosting mechanism.

> :warning: For this integration package to work properly, you need version 1.1 of ASP.NET Core

### Installation

Add `StructureMap.AspNetCore` to your **project.json**:

```json
"dependencies": {
  "StructureMap.AspNetCore": "<version>"
}
```

### Usage

The package adds the `UseStructureMap` extension method to `IWebHostBuilder`. Calling this method will instruct the ASP.NET Core host to
create a StructureMap `Registry` and optionally let the user configure it using a `Startup.ConfigureContainer(Registry)` method.

### Example

```csharp
using System.IO;
using Microsoft.AspNetCore.Hosting;

public class Program
{
    public static void Main(string[] args)
    {
        var host = new WebHostBuilder()
            .UseContentRoot(Directory.GetCurrentDirectory())
            .UseStartup<Startup>()
            .UseIISIntegration()
            .UseStructureMap()
            .UseKestrel()
            .Build();

        host.Run();
    }
}
```

The runtime will then look for a `ConfigureContainer` method on the specified `Startup` class:

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Configure the ASP.NET specific stuff.
    }

    public void ConfigureContainer(Registry registry)
    {
        // Use StructureMap-specific APIs to register services in the registry.
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
    {
    }
}
```


## StructureMap.Microsoft.DependencyInjection

(Formerly known as StructureMap.Dnx)

Adds StructureMap support for [Microsoft.Extensions.DependencyInjection](https://github.com/aspnet/DependencyInjection)

### Installation

Add `StructureMap.Microsoft.DependencyInjection` to your **project.json**:

```json
"dependencies": {
  "StructureMap.Microsoft.DependencyInjection": "<version>"
}
```

### Usage

The package contains a single, public extension method, `Populate`.
It's used to populate a StructureMap container using a set of `ServiceDescriptors` or an `IServiceCollection`.

#### Example

```csharp
using System;
using Microsoft.Extensions.DependencyInjection;
using StructureMap;

public class Startup
{
    public IServiceProvider ConfigureServices(IServiceCollection services)
    {
        services.AddMvc();
        services.AddWhatever();

        var container = new Container();
        
        // You can populate the container instance in one of two ways:
        
        // 1. Use StructureMap's `Configure` method and call
        //    `Populate` on the `ConfigurationExpression`.
        
        container.Configure(config =>
        {
            // Register stuff in container, using the StructureMap APIs...

            config.Populate(services);
        });
        
        // 2. Call `Populate` directly on the container instance.
        //    This will internally do a call to `Configure`.
        
        // Register stuff in container, using the StructureMap APIs...

        // Here we populate the container using the service collection.
        // This will register all services from the collection
        // into the container with the appropriate lifetime.
        container.Populate(services);

        // Finally, make sure we return an IServiceProvider. This makes
        // ASP.NET use the StructureMap container to resolve its services.
        return container.GetInstance<IServiceProvider>();
    }
}
```
