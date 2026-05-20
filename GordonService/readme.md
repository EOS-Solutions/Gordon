# Gordon Service

## Overview

Gordon Service is a Windows Service that enables seamless communication between installed Business Central Services and the tools from the Gordon Suite. It acts as a critical bridge layer, allowing Gordon tools (such as Gordon for VS Code, Gordon PowerShell, and other developer utilities) to interact with Business Central instances. This service exposes a RESTful API that allows access to Business Central administration cmdlets without need to grant a developer full administrative access to a server.

## Key Capabilities

- Service-based architecture for reliable, always-available connectivity
- API key-based authentication and access isolation
- Machine-wide installation for centralized management
- Configuration management for Business Central instances
- Support for automation and scripting workflows
- Single entry-point with a well-defined API surface for all BC services running on a machine.

## Technical Architecture

Gordon Service is built on **.NET with Kestrel web server**, providing a lightweight and high-performance communication layer:

- **Web Server**: Runs on .NET's built-in Kestrel web server
- **Default Port**: 9462 (customizable via standard Kestrel configuration)
- **Requirements**: Must run as local administrator to communicate with Business Central services
- **Configuration**: Minimal setup required - works with defaults out of the box

## Service Clients

Gordon Service will spin up a **service client** (using PowerShell) to connect to the actual BC service. Each BC service will have it's own **service client** running that will be kept alive for a given time and then removed. This allows isolated access on a per-service basis. This will also allow developers administrative access to the service only, not to the entire machine. 

This also makes sure that you can access multiple difference BC versions running on the same server from a single entry point.

If you use API keys, you can also limit which key has access to which BC service. See [Configuration](configuration.md) for how to configure API keys.

The choice which service client is used for which service can be configured (see [Configuration](configuration.md)), but will mostly be left as the default for most users. That is:
- a .NET 8 client using PS7 for BC24+
- a .NET Framework 4.8 client using PS5 for BC23 and below.

You can find the service client executables Gordon Service uses in the subfolder `PsHost`.

## Installation

Gordon Service must be installed machine-wide using the [EOS Gordon Installer](https://eos-solutions.github.io/Gordon/). For detailed installation instructions, refer to the [Gordon Service Installation Guide](https://docs.eos-solutions.it/en/docs/dev-tools/gordon-service.html).

## Configuration

For guidance on configuring Gordon Service and setting up connections to your Business Central instances, see the [Configuration](configuration.md).

## Part of Gordon Suite

Gordon Service is one component of the comprehensive Gordon Suite of development tools for Business Central. Most of Gordon's operations (the clients below) will require you to have Gordon Service running on the target machine.

- **Gordon UI** - A desktop application to manage your BC services and apps.
- **Gordon for VS Code** - Visual Studio Code extension
- **Gordon PowerShell** - PowerShell module for automation
- **Gordon Installer** - Installation and management utility
- **Gordon Service** - Core communication layer (this component)

---

For more information, visit the [Gordon Suite Documentation](https://docs.eos-solutions.it/en/docs/dev-tools/gordon-introduction.html)