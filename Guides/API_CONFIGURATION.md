# API Configuration - Single Source of Truth (SSOT)

## Overview

This document describes the centralized API configuration system that allows you to easily manage the backend REST API base URL from a single location.

## Problem Solved

Previously, the API base URL was hardcoded in multiple locations throughout the codebase:
- `CameraApiService.cpp` - Camera management operations
- `AuthDialog.cpp` - User authentication
- `UserProfileWidget.cpp` - User profile operations  
- `VpnWidget.cpp` - WireGuard VPN configuration

This made it difficult to change the API endpoint when switching between development, staging, and production environments.

## Solution

All API base URL references now use the `ConfigManager` class as a single source of truth. The API base URL is stored in the configuration file and can be easily modified.

## Configuration

### Method 1: Configuration File

The API base URL is stored in the `config.json` file in your application data directory:

**Windows:** `%LOCALAPPDATA%\ViscoConnect\config.json`

Example configuration:
```json
{
    "autoStart": false,
    "echoServerEnabled": true,
    "echoServerPort": 7777,
    "apiBaseUrl": "http://54.225.63.242:8086",
    "cameras": [
        // ... camera configurations
    ]
}
```

### Method 2: Programmatic Configuration

You can also change the API base URL programmatically:

```cpp
#include "ConfigManager.h"

// Get current API base URL
QString currentUrl = ConfigManager::instance().getApiBaseUrl();

// Set new API base URL
ConfigManager::instance().setApiBaseUrl("https://your-new-api-server.com:8080");
```

## Default Configuration

The default API base URL is: `http://54.225.63.242:8086`

This maintains backward compatibility with existing installations.

## Affected Components

The following components automatically use the centralized API configuration:

1. **CameraApiService** - All camera management operations (create, update, delete, status updates)
2. **AuthDialog** - User login and authentication  
3. **UserProfileWidget** - User profile fetching
4. **VpnWidget** - WireGuard configuration generation

## Runtime Configuration Updates

The system supports runtime configuration updates. When the API base URL is changed:

1. The `ConfigManager` emits a `configChanged()` signal
2. `CameraApiService` automatically updates its base URL
3. A new connectivity check is performed with the updated URL
4. All subsequent API requests use the new base URL

## Usage Examples

### Changing API Server for Development

To switch to a development API server, update your `config.json`:

```json
{
    "apiBaseUrl": "http://localhost:3000",
    // ... other settings
}
```

### Switching to HTTPS

To use a secure HTTPS endpoint:

```json
{
    "apiBaseUrl": "https://api.yourserver.com",
    // ... other settings  
}
```

### Using Different Ports

To use a different port:

```json
{
    "apiBaseUrl": "http://192.168.1.100:9000",
    // ... other settings
}
```

## Configuration Management

### Validation

The system performs basic validation:
- Empty URLs are rejected
- Configuration changes are logged
- Invalid configurations fall back to default values

### Logging

All API base URL changes are logged with the following format:
```
[Config] API base URL changed to https://new-server.com:8080
[CameraApiService] API base URL updated from http://old-server.com to https://new-server.com:8080
```

### Backup and Recovery

- The configuration is automatically saved when changed
- The system creates a default configuration if none exists
- Invalid configurations are logged and use safe defaults

## Troubleshooting

### Configuration Not Taking Effect

1. Check that the `config.json` file is in the correct location
2. Verify the JSON syntax is valid
3. Check the application logs for configuration errors
4. Restart the application if needed

### API Connection Issues

1. Verify the new API base URL is accessible
2. Check network connectivity to the new server
3. Ensure the API server is running on the specified port
4. Check firewall settings if using a different server

### Finding Configuration File

The configuration file location can be found programmatically:

```cpp
QString configPath = ConfigManager::instance().getConfigFilePath();
qDebug() << "Config file location:" << configPath;
```

## Migration from Hardcoded URLs

If you're upgrading from a version with hardcoded URLs:

1. The application will automatically create a default configuration
2. Existing installations will continue to work with the default URL
3. No manual migration is required
4. You can immediately start using the new configuration system

## Security Considerations

- Store sensitive configuration in secure locations
- Use HTTPS endpoints in production environments  
- Regularly update API endpoints to maintain security
- Monitor configuration changes in production systems

## Future Enhancements

Potential future improvements to the configuration system:

- UI-based configuration editor
- Environment variable support
- Multiple API endpoint configurations
- Configuration validation and testing
- Automatic failover between endpoints
