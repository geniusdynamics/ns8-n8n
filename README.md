# NS8 n8n Module

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![n8n Version](https://img.shields.io/badge/n8n-2.48.4-orange.svg)](https://github.com/n8n-io/n8n)

n8n allows you to build flexible workflows focused on deep data integration. This NS8 module provides a complete containerized deployment with PostgreSQL, Redis, and Traefik integration.

## 🏗️ Architecture

The module consists of multiple services orchestrated by systemd:

- **n8n.service**: Main pod service that manages network connectivity
- **n8n-pgsql.service**: PostgreSQL database (v15.5-bookworm)
- **n8n-redis.service**: Redis cache (v7.2.3-bookworm) 
- **n8n-server.service**: n8n application server (v2.48.4)

## 📦 Container Images

The build process uses the following container images:

- `docker.io/library/redis:7.2.3-bookworm` - Redis database
- `docker.io/library/postgres:15.5-bookworm` - PostgreSQL database
- `docker.io/n8nio/n8n:2.48.4` - n8n workflow automation
- `docker.io/library/node:lts` - Node.js build environment

## 🔧 Build Process

The `build-images.sh` script creates the module image using Buildah:

1. **UI Compilation**: Builds the Vue.js frontend with Node.js
2. **Image Assembly**: Adds imageroot directory and compiled UI
3. **Configuration**: Sets up entrypoint, labels, and container requirements
4. **Publishing**: Pushes to GitHub Container Registry

Key build features:
- Reuses `nodebuilder-ns8-n8n` container for faster builds
- Configures Traefik integration with route authorization
- Sets rootless container with TCP port requirements
- Supports CI/CD with GitHub Actions

## 🚀 Installation

Instantiate the module with:

```bash
add-module ghcr.io/geniusdynamics/n8n:latest 1
```

The output will return the instance name:

```json
{"module_id": "n8n", "image_name": "n8n", "image_url": "ghcr.io/geniusdynamics/n8n:latest"}
```

## ⚙️ Configuration

Assuming the n8n instance is named `n8n1`, configure with:

```bash
api-cli run module/n8n1/configure-module --data '{
    "host": "n8n.domain.com",
    "lets_encrypt": false,
    "http2https": true
}'
```

### Configuration Parameters

- `host`: Traefik host URL for the project
- `lets_encrypt`: Enable/disable Let's Encrypt (default: false)
- `http2https`: Enable HTTP to HTTPS redirect (default: true)

## 🔍 Service Management

### Systemd Services

All services are managed by systemd with the following characteristics:

- **Restart Policy**: `always` for high availability
- **Timeout**: 70 seconds for graceful shutdown
- **Dependencies**: Proper service ordering (PostgreSQL → Redis → n8n)
- **Health Checks**: Built-in container health monitoring

### Service Dependencies

```
n8n.service (pod)
├── n8n-pgsql.service (database)
├── n8n-redis.service (cache)
└── n8n-server.service (application)
```

## 🌐 Network Configuration

The module publishes the n8n web interface on:

- **Internal**: Port 5678 within the pod
- **External**: `127.0.0.1:${TCP_PORT}` via Traefik

Test the service:

```bash
curl http://127.0.0.1/n8n/
```

## 🔄 Smarthost Integration

The module automatically discovers smarthost settings from Redis:

- `bin/discover-smarthost` refreshes `state/smarthost.env`
- `events/smarthost-changed/10reload_services` handles configuration updates
- Ensures email settings stay synchronized with NS8 core

## 🧪 Testing

Test the module using the provided test script:

```bash
./test-module.sh <NODE_ADDR> ghcr.io/geniusdynamics/n8n:latest
```

Tests are implemented using [Robot Framework](https://robotframework.org/).

## 🌍 Internationalization

The UI supports multiple languages through Weblate:

- **Supported Languages**: English, German, Spanish, Basque, Italian
- **Translation Platform**: [Weblate](https://hosted.weblate.org/projects/ns8/)

To enable translation updates:
1. Add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup)
2. Request addition to NS8 Weblate project

## 🔄 Updates

Update the module to the latest version:

```bash
api-cli run update-module --data '{
  "module_url": "ghcr.io/geniusdynamics/n8n:latest",
  "instances": ["n8n1"],
  "force": true
}'
```

## 🗑️ Uninstallation

Remove the instance (data will be preserved unless `--no-preserve` is used):

```bash
remove-module --no-preserve n8n1
```

## 📁 Volume Storage

The module uses persistent volumes for:

- `db_storage`: PostgreSQL data directory
- `n8n-redis-data`: Redis data with AOF enabled
- `n8n_data`: n8n user data and workflows

## 🔒 Security Features

- **Rootless Containers**: All services run without root privileges
- **Network Isolation**: Services communicate within a private pod
- **Traefik Integration**: Secure HTTPS termination and routing
- **Environment Files**: Sensitive configuration stored separately

## 📄 License

This module is licensed under GPL-3.0-or-later. See [LICENSE](LICENSE) for details.

## 🤝 Contributing

Contributions are welcome! Please ensure all changes:
- Follow the existing code style
- Include appropriate tests
- Update documentation as needed

