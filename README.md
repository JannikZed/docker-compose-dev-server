# CI Services Stack

This repository contains a Docker Compose setup for running a complete CI/CD stack with the following services:

- Traefik (v2.10) - Reverse Proxy with SSL termination
- Jenkins (2.489-jdk17) - Continuous Integration Server
- SonarQube (LTS) - Code Quality Analysis
- Docker-in-Docker - For running Docker commands inside Jenkins

## Prerequisites

- Docker Engine 20.10+
- Docker Compose v2+
- OpenSSL (for generating certificates)

## Quick Start

1. Generate self-signed certificates:
```bash
# Create certs directory if it doesn't exist
mkdir -p certs

# Generate self-signed certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout certs/server.key \
  -out certs/server.crt \
  -subj "/CN=localhost/O=My Company/C=US"
```

2. Start the services:
```bash
docker-compose up -d
```

## Service URLs

- Jenkins: https://localhost/jenkins
- SonarQube: https://localhost/sonarqube
- Traefik Dashboard: https://localhost/dashboard/

## Configuration Details

### Traefik

- Uses custom SSL certificates
- Configuration in `traefik.yml` and dynamic configuration in `certs/tls.yml`
- Handles routing and SSL termination for all services

### Jenkins

- Runs with Docker-in-Docker support
- Accessible under the `/jenkins` context path
- Resource limits: 4GB RAM, 2 CPUs

### SonarQube

- Runs in LTS version
- Accessible under the `/sonarqube` context path
- Resource limits: 2GB RAM, 1 CPU
- Uses named volumes for data persistence

## Volume Management

The stack uses several named volumes for data persistence:

- `jenkins_home`: Jenkins configuration and data
- `sonarqube_data`: SonarQube data
- `sonarqube_extensions`: SonarQube plugins
- `sonarqube_logs`: SonarQube logs

## Security Notes

- Custom SSL certificates are used for HTTPS
- Traefik dashboard is secured
- All services are isolated in their own network
- Sensitive files (certificates) are git-ignored

## Maintenance

### Backup Volumes

To backup the data, you can use Docker's volume backup feature:

```bash
docker run --rm -v jenkins_home:/source:ro -v $(pwd):/backup alpine tar czf /backup/jenkins_backup.tar.gz -C /source ./
```

### Updating Services

To update services to their latest versions:

```bash
docker-compose pull
docker-compose up -d
```

## Troubleshooting

### Certificate Issues

If you encounter certificate warnings in your browser:
- For development: Add the self-signed certificate to your system's trust store
- For production: Replace the certificates with valid ones from a trusted CA

### Permission Issues

If you encounter permission issues with SonarQube:
- Ensure the volumes have correct permissions
- The service runs with user ID 1000:1000

## Development

### Adding New Services

1. Add the service configuration to `docker-compose.yml`
2. Configure Traefik labels for routing
3. Update the README with new service details

### Local Development

For local development, you can use the `.env` file to override configurations:

```bash
cp .env.example .env
# Edit .env with your settings