# Gordon Service Configuration

## Overview

Gordon Service requires minimal configuration to get started. Most settings use sensible defaults that work out of the box for standard Business Central deployments.

## Configuration Framework

All Gordon Service configuration is performed using the **standard .NET configuration framework**. This provides a flexible, industry-standard approach to managing application settings.

Gordon Service **does not require configuration** for basic operation. It works with default settings immediately after installation.

When customization is needed, Gordon Service uses the standard [.NET Configuration system](https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration) to manage all settings. This framework supports:

Gordon Service respects the standard .NET configuration provider chain. Settings can be configured through:

1. **appsettings.json** - Default configuration file
3. **Environment variables** - System environment variables
4. **Command-line arguments** - Runtime parameters

For detailed information on how the .NET configuration system works, refer to the [Microsoft .NET Configuration Documentation](https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration).

## Supported Configuration settings

- `MaxClientIdleTime`: Gordon service uses the concept of [Service Clients](readme.md#service-clients) for connecting to a BC service. This setting specifies how long a service client can be idle before it is removed and it's resources freed up. Default: 10min.

- `ServiceClientMapPath`: Specifies a path to a file with the service client map. This allows configuring which BC service uses which service client executable. Specifically: this will use a .NET8 client for BC24+ and a .NET Framework 4.8 client for BC23 and below. You need to modify this only for specific cases, the default should work fine.

- `AuditLogLevel`: Recent Gordon Service versions support emitting an audit log (which API key has executed which action when on what service). This is disabled by default. Audit messages are written to the default logging infrastructure (see [Logging](logging.md)). Use this setting to specify what to log. Valid values are: 
    - `None`: nothing is logged.
    - `SuccessOnly`: only successful requests will be logged.
    - `All`: all requests will be logged.

- `ApiKeys`: This is an array of API keys that are valid for this Gordon Service instance. If no API key is defined, Gordon Service will not enforce authentication and anyone can do anything. If at least on key is defined, the API key must be provided on each request. See the Gordon swagger document for technical details. Each API key must be defined as a JSON object in this array with the following properties:
    - `Key`: the API key.
    - `Description`: an optional description for the API key.
    - `Enabled`: a boolean specifying whether the key is enabled.
    - `EnabledServiceRegex`: an array of strings. Each string is a regular expression that specifies which services this API key can be used with. If nothing is specified, all services are valid. Otherwise all services where it's name at least *one* of the regular expressions match are allowed.

- `ApiKeysFile`: a path (local file system) or a URL (HTTP) to a JSON file that contains an array of ApiKeys (see above). This allows to define the API keys in a file external to the BC server. Any API keys defined here are added to the ones specified via `ApiKeys`.

### Customizing Kestrel Configuration

To customize the port or other Kestrel web server settings, modify the configuration using standard .NET Kestrel configuration options. See the [Kestrel Configuration Documentation](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/endpoints) for available options.

## Technical Architecture

### Web Server

Gordon Service runs on **.NET's Kestrel web server**, a high-performance cross-platform HTTP server built into .NET. For more information about Kestrel capabilities and advanced settings, refer to the [Microsoft Kestrel documentation](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel).

### Port Configuration

- **Default Port**: `9462`
- **Customization**: The port can be customized using the standard [.NET Kestrel configuration](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/endpoints)

Port configuration follows the standard .NET configuration framework. You can override the default port through any of the supported configuration sources (JSON files, environment variables, etc.).

## Security Requirements

### Administrator Privileges

⚠️ **Important**: Gordon Service must run as **local administrator** to communicate with any Business Central services on the machine.

This is required because:
- Business Central instances require elevated privileges for service communication
- AppPool isolation and management operations need administrative access
- Performance monitoring and diagnostics require elevated permissions

Ensure the service account running Gordon Service has local administrator privileges on the machine.

## Next Steps

For detailed installation and setup instructions, refer to the [Gordon Service Installation Guide](https://docs.eos-solutions.it/en/docs/dev-tools/gordon-service/gordon-service-configuration.html).