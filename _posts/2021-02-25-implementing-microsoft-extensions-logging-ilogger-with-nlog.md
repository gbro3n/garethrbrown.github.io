---
public: true
layout: post
title: "Implementing Microsoft.Extensions.Logging.ILogger with NLog"
date: 2021-02-25 00:00 +0000
tags: nlog net-core
---

I’ve designed this `ILogger`implementation to solve an issue I encountered today where the code I was working was wasn’t logging errors when the error occurred in a background thread. It was failing silently because unless you configure NLog specifically to throw exceptions, it won’t.

The principle issue in my case was that the usual `ILogger<T>` instance instantiated in the class constructor wasn’t available because the controller object on the main thread had disposed before the thread had completed. Fair enough. I then tried creating an instance of `ILogger<T>` via a service locator within the thread. And that failed silently too (I’m not sure of what was going on internally here but I’m wondering if it’s to do with the Logger being typed to the class which is disposed, would need to check the framework source code here).

So I opted to create a class that implements `Microsoft.Extensions.Logging.ILogger`, and register it as a singleton that can be used everywhere and will not be disposed with the container object.

[So we loose the capabilities](https://stackoverflow.com/questions/53825831/asp-net-core-api-why-does-logger-type-match-controller-class) of `ILogger<T>` which would allow the logger to record information about the type, but to me this is a simplification and has practical benefits when working with async code.

## The code

Package references that may be required depending on the .NET core framework version you are targeting.

```xml
<PackageReference Include="Microsoft.Extensions.Logging" Version="3.1.5" />  
<PackageReference Include="NLog" Version="4.5.10" />  
<PackageReference Include="NLog.Web.AspNetCore" Version="4.7.0" />
```

## The NLogLogger class it’s self

```csharp
using Microsoft.Extensions.Logging;
using NLog;
using System;

namespace ANamespace
{
    public class NLogLogger : Microsoft.Extensions.Logging.ILogger
    {
        private NLog.ILogger NLogLoggerInstance { get; }

        private class DisposableStub : IDisposable
        {
            public void Dispose()
            {
                // Do nothing
            }
        }

        public NLogLogger()
        {
            NLogLoggerInstance = LogManager.LoadConfiguration("NLog.config").GetCurrentClassLogger();
        }

        public IDisposable BeginScope<TState>(TState state)
        {
            // Not implementing this feature

            return new DisposableStub();
        }

        public bool IsEnabled(Microsoft.Extensions.Logging.LogLevel logLevel)
        {
            if (logLevel == Microsoft.Extensions.Logging.LogLevel.Trace)
            {
                return NLogLoggerInstance.IsTraceEnabled;
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.Debug)
            {
                return NLogLoggerInstance.IsDebugEnabled;
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.Information)
            {
                return NLogLoggerInstance.IsInfoEnabled;
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.Warning)
            {
                return NLogLoggerInstance.IsWarnEnabled;
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.Error)
            {
                return NLogLoggerInstance.IsErrorEnabled;
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.Critical)
            {
                return NLogLoggerInstance.IsFatalEnabled;
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.None)
            {
                return false;
            }
            else
            {
                throw new InvalidOperationException($"Unhandled type of {nameof(Microsoft.Extensions.Logging.LogLevel)}");
            }
        }

        public void Log<TState>(Microsoft.Extensions.Logging.LogLevel logLevel, EventId eventId, TState state, Exception exception, Func<TState, Exception, string> formatter)
        {
            if (logLevel == Microsoft.Extensions.Logging.LogLevel.Trace)
            {
                NLogLoggerInstance.Trace(formatter(state, exception));
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.Debug)
            {
                NLogLoggerInstance.Debug(formatter(state, exception));
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.Information)
            {
                NLogLoggerInstance.Info(exception, formatter(state, exception));
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.Warning)
            {
                NLogLoggerInstance.Warn(exception, formatter(state, exception));
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.Error)
            {
                NLogLoggerInstance.Error(exception, formatter(state, exception));
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.Critical)
            {
                NLogLoggerInstance.Fatal(exception, formatter(state, exception));
            }
            else if (logLevel == Microsoft.Extensions.Logging.LogLevel.None)
            {
                NLogLoggerInstance.Info(formatter(state, exception));
            }
        }
    }
}
```

Registering as a service for dependency injection in your ASP.NET core app:

```csharp
services.AddSingleton<Microsoft.Extensions.Logging.ILogger>(new ANamespace.NLogLogger());
```