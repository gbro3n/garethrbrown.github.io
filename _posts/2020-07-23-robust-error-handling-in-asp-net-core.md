---
public: true
layout: post
title: "Robust Error Handling in ASP.NET Core"
date: 2020-07-23 00:00 +0000
tags: netcore errors
---

Error handling in .NET Core is great, but there are some pitfalls if not properly set up that can result in unexpected behaviour. For example - HTTP status codes that appear incorrect, or some categories of errors not being trapped.

I revisited my error handling code because of a few issues I was experiencing:

- Errors that occurred before the request had made it inside controller code were returning HTTP 405 (Method not allowed) to the caller / browser.
- Errors in API controllers were returning HTML in the response to the caller, when a simple text or JSON response would be more appropriate.

The main reference documentation is here:

[https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-3.1](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/error-handling?view=aspnetcore-3.1)

## The core issues

### Changing [HttpGet] to [Route]

I was using the `[HttpGet("error/{statusCode:int}")]` HTTP verb attribute on my error controller action. This meant that where an error was thrown and the original action was an HTTP POST, `UseStatusCodePagesWithReExecute` re-executes the pipeline but is not allowed to call the `Error` action because `[HttpGet("error/{statusCode:int}")]` requires that the action is only available in the context of an HTTP POST. To resolve this, I needed to use `[Route("error/{statusCode:int}")]` instead.

Startup.cs

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env, IConfiguration configuration)
{
    // ....
    
    // Ensures graceful handling of non exception errors (404's etc)

    app.UseStatusCodePagesWithReExecute("/error/{0}");

    // Ensures custom error pages for unhandled exceptions

    app.UseExceptionHandler("/error/500");
```

### Checking paths in HttpContext.Features to determine if the original request was for an API controller

Using properties on the variables below ...

```csharp
var statusCodeFeature = HttpContext.Features.Get<IStatusCodeReExecuteFeature>();

var exceptionDataFeature = HttpContext.Features.Get<IExceptionHandlerPathFeature>();
```

We can later check the path for an `/api` prefix and return the appropriate response.   
    
```csharp
bool isApiPath = (statusCodeFeature?.OriginalPath != null && statusCodeFeature.OriginalPath.StartsWith("/api", StringComparison.InvariantCultureIgnoreCase)) ||
                 (exceptionDataFeature?.Path != null && exceptionDataFeature.Path.StartsWith("/api", StringComparison.InvariantCultureIgnoreCase));

if (isApiPath)
{
    actionResult = Content($"The request could not be processed ({statusCode.ToString(CultureInfo.InvariantCulture)})");
}
else
{
    ViewBag.StatusCode = statusCode;
    actionResult = View();
}
```

## Full Source Code

ErrorController.cs

```csharp
using System;
using System.Globalization;
using Microsoft.AspNetCore.Diagnostics;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Logging;

namespace ExampleApp.Controllers
{
    public class ErrorController : Controller
    {
        private readonly IConfiguration _configuration;
        private readonly ILogger _logger;

        public ErrorController(IConfiguration configuration, ILogger logger) : base(configuration)
        {
            _configuration = configuration;
            _logger = logger;
        }

        // Must be [Route] attribute and not HttpGet (for example) as if the original erroring action does
        // not match then this results in an 40t status code which is confusing for the recipient

        [Route("error/{statusCode:int}")]
        public IActionResult Error(int statusCode)
        {
            var statusCodeFeature = HttpContext.Features.Get<IStatusCodeReExecuteFeature>();

            var exceptionDataFeature = HttpContext.Features.Get<IExceptionHandlerPathFeature>();

            string logString = null;

            if (statusCodeFeature != null)
            {
                logString += $"Status Code: {statusCode} Status Code Path Base: [{statusCodeFeature.OriginalPathBase}], Status Code Path: [{statusCodeFeature.OriginalPath}], Status Code Query String: [{statusCodeFeature.OriginalQueryString}]";
            }

            if (exceptionDataFeature != null)
            {
                if (logString != null)
                {
                    logString += Environment.NewLine;
                }

                logString += $"Status Code: {statusCode} Exception Path: [{exceptionDataFeature.Path}], Exception: [{exceptionDataFeature.Error}], Exception Message: [{exceptionDataFeature.Error.Message}], Exception Stack Trace: [{exceptionDataFeature.Error.StackTrace}]";
            }

            if (statusCodeFeature == null && exceptionDataFeature == null)
            {
                // Unsure if it's possible for both to be null but log something  in case

                logString += $"Status Code: {statusCode}. Both {nameof(statusCodeFeature)} and {nameof(exceptionDataFeature)} were null in {nameof(ErrorController)} {nameof(Error)}";
            }

            if (statusCode >= 400 && statusCode <= 499)
            {
                // Log warning to ensure captured in production logging. Although possibly not our
                // error, something is not communicating with our app as expected and so this warrants
                // the warn flag.

                _logger.LogWarning(logString);
            }
            else
            {
                _logger.LogError(logString);
            }

            IActionResult actionResult;

            // Check both statusCodeFeature and exceptionDataFeature for path

            bool isApiPath = (statusCodeFeature?.OriginalPath != null && statusCodeFeature.OriginalPath.StartsWith("/api", StringComparison.InvariantCultureIgnoreCase)) ||
                             (exceptionDataFeature?.Path != null && exceptionDataFeature.Path.StartsWith("/api", StringComparison.InvariantCultureIgnoreCase));

            if (isApiPath)
            {
                actionResult = Content($"The request could not be processed ({statusCode.ToString(CultureInfo.InvariantCulture)})");
            }
            else
            {
                ViewBag.StatusCode = statusCode;

                actionResult = View();
            }

            return actionResult;
        }

        // Test actions to trigger basic use case scenarios

        [HttpGet("error/test-throw-404")]
        public IActionResult Throw404() => StatusCode(404);

        [HttpGet("error/test-throw-422")]
        public IActionResult Throw422() => StatusCode(422);

        [HttpGet("error/test-throw-500")]
        public IActionResult Throw500() => StatusCode(500);

        [HttpGet("error/test-throw-exception")]
        public IActionResult ThrowException()
        {
            throw new InvalidOperationException("Test exception");
        }
    }
}
```