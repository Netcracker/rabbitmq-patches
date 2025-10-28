# RabbitMQ Patches Repository - Copilot Instructions

## Purpose & Architecture

This repository manages patches for RabbitMQ server builds and automated Docker image creation. The core workflow involves:

1. **Patch Storage**: Version-specific patches stored in `rabbitmq-patches/v{VERSION}/` directories
2. **Automated Building**: GitHub Actions workflows that checkout RabbitMQ source, apply patches, build binaries, and create Docker images  
3. **Multi-Image Strategy**: Builds both base RabbitMQ images and management-enabled variants
4. **GHCR Distribution**: Pushes patched images to GitHub Container Registry

## Critical Directory Structure

```
rabbitmq-patches/
├── v4.1.2/
│   └── *.patch           # Git patches for specific RabbitMQ versions
├── v4.2.x/               # Future version patches (add as needed)
└── ...
.github/workflows/
├── qb-integration-build.yml    # Full integration workflow (recommended)
├── build-rabbitmq-patched.yaml # Basic build workflow  
└── cla.yaml                    # CLA enforcement
```

## Key Workflows

### Primary Build: `qb-integration-build.yml` 
- **Purpose**: Complete RabbitMQ build with Docker image creation
- **Trigger**: Manual dispatch with version inputs
- **Key Parameters**:
  - `rabbitmq-tag`: RabbitMQ version to patch (e.g., "v4.1.2")
  - `tag`: Output tag for patched version (e.g., "v4.1.2-patched")
  - `docker-library-rabbitmq`: Docker library version ("4.1" or "4.2")
- **Outputs**: 
  - `ghcr.io/{owner}/rabbitmq-base:{tag}` - Base patched image
  - `ghcr.io/{owner}/rabbitmq:{tag}-management` - Management-enabled image
  - Release assets with compiled binaries

### Patch Application Process
The workflows use this exact pattern for applying patches:
```bash
for p in ./rabbitmq-patches/rabbitmq-patches/${{ inputs.rabbitmq-tag }}/*.patch; do
  echo "Applying $p…"
  git apply -p1 --ignore-space-change --ignore-whitespace "$p"
done
```

## Development Patterns

### Adding New Version Patches
1. Create new directory: `rabbitmq-patches/v{NEW_VERSION}/`
2. Add `.patch` files using `git format-patch` or `git diff` format
3. Patches must be compatible with `git apply -p1` command
4. Test with workflow dispatch using new version tag

### Docker Image Modification Strategy
The workflows modify official `docker-library/rabbitmq` Dockerfiles by:
- Replacing wget downloads with local COPY of built binaries
- Injecting environment variables (`RABBITMQ_HOME`, `PATH`)
- Using sed patterns to replace download blocks with local file operations

### Version Management
- **Input versions**: Use exact RabbitMQ tags (e.g., "v4.1.2")
- **Output tags**: Append "-patched" suffix (e.g., "v4.1.2-patched")
- **Docker library versions**: Match major.minor (e.g., "4.1" for v4.1.x)

## Prerequisites & Dependencies

### Build Environment (Ubuntu)
- **Erlang/Elixir**: Installed via `ppa:rabbitmq/rabbitmq-erlang`
- **Build tools**: `build-essential`, `debhelper`, `xmlto`
- **XML processing**: `libxslt` (via snap)
- **Specific Erlang modules**: See workflow install steps for complete list

### External Dependencies
- **Netcracker actions**: `netcracker/qubership-workflow-hub/actions/*`
- **Docker registries**: GitHub Container Registry (ghcr.io)
- **Source repositories**: 
  - `rabbitmq/rabbitmq-server` - Source code
  - `docker-library/rabbitmq` - Official Dockerfiles

## Common Operations

### Testing Patch Application Locally
```bash
# Checkout RabbitMQ source
git clone --branch v4.1.2 https://github.com/rabbitmq/rabbitmq-server.git
cd rabbitmq-server

# Apply patches
for p in /path/to/rabbitmq-patches/v4.1.2/*.patch; do
  git apply -p1 --ignore-space-change --ignore-whitespace "$p"
done
```

### Running Builds
Use GitHub Actions manual dispatch with appropriate version parameters. The `qb-integration-build.yml` workflow is the primary build process.

### Troubleshooting Build Issues
- Check patch compatibility with target RabbitMQ version
- Verify Erlang/Elixir version compatibility
- Review Docker build logs for Dockerfile sed modifications
- Ensure GHCR permissions for image pushes

## Integration Points

- **CLA enforcement**: Managed by Netcracker's CLA assistant
- **Release management**: Automated via Netcracker workflow hub actions
- **Container registry**: GHCR with repository owner namespace
- **Upstream sources**: Direct integration with official RabbitMQ and Docker repositories