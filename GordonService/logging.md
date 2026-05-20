# Gordon Service Logging

## Overview

Gordon Service logging is built on the **standard .NET logging infrastructure**, providing flexible, extensible, and industry-standard diagnostic capabilities. Out of the box, the service supports multiple logging destinations to meet different operational needs.

There are two different components at play here. Logging for **Gordon Service** (the core API service) and logging for each **service client**. This article applies to both components the same way, with one key difference only: the **service clients** do not support logging to application insights, they will log to file exclusively. The rest of this documentation applies to both. See more at [service clients](readme.md#service-clients)

By default both service clients will write it's log to a file called `ps-host.log` that lives in the same folder, while the service will write it's log to `log.txt` in the root folder of the service. Both can be changed to anything you like.

## .NET Logging Framework

Gordon Service uses the standard [.NET Logging abstraction](https://learn.microsoft.com/en-us/dotnet/core/extensions/logging), which provides:

- **Multiple log levels**: Trace, Debug, Information, Warning, Error, Critical, and None
- **Structured logging**: Semantic logging with structured data
- **Provider architecture**: Pluggable logging providers for different outputs
- **Configuration flexibility**: Configure logging through standard .NET configuration
- **Performance**: Efficient filtering and async support

For comprehensive information on how the .NET logging framework works, refer to the [Microsoft .NET Logging Documentation](https://learn.microsoft.com/en-us/dotnet/core/extensions/logging).

## Supported Logging Providers

Gordon Service includes built-in support for the following logging destinations:

### Console Logging
Logs output directly to the console. Useful for development and debugging. Automatically included in the standard .NET logging framework. This only works when running Gordon Service from the console.

### File Logging
Gordon Service uses **[Nreco.FileLogger](https://github.com/nreco/logging)** for file-based logging. This provider writes logs to files on disk, supporting:
- Rolling log files (file rotation by date or size)
- Customizable log file naming and locations
- Structured log formatting

For configuration options and advanced features, refer to the [Nreco.FileLogger Documentation](https://github.com/nreco/logging).

### Application Insights (Gordon Service only)
Gordon Service supports **Azure Application Insights** for cloud-based monitoring and diagnostics. Application Insights provides:
- Centralized log aggregation and analysis
- Performance monitoring and metrics
- Alert capabilities
- Integration with other Azure services

To enable Application Insights logging, configure the [Application Insights .NET SDK](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) through standard .NET configuration.

This is not available for the service clients.

## Configuration

Logging is configured using the standard [.NET configuration framework](https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration). Configure logging providers and levels through:

1. **appsettings.json** - Default logging configuration
2. **appsettings.{Environment}.json** - Environment-specific logging settings
3. **Environment variables** - Runtime logging overrides
4. **Command-line arguments** - Launch-time configuration

### Example Configuration

Logging configuration typically appears in your `appsettings.json` file under the `Logging` section:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "GordonService": "Debug"
    },
    "Console": {
      "IncludeScopes": true
    },
    "File": {
      "Path": "logs/gordon-service.log",
      "MinLevel": "Information"
    }
  }
}
```

For detailed configuration options, refer to the [.NET Logging Configuration Documentation](https://learn.microsoft.com/en-us/dotnet/core/extensions/logging-configuration).