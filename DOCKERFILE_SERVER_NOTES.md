# Dockerfile.server Implementation Notes

## Overview

This document provides implementation notes for the `Dockerfile.server` that was created to satisfy the requirement of building a server-only Docker image for Label Studio.

## Requirements Met

The Dockerfile.server implements all the steps specified in the README.md for local development:

1. ✅ Install poetry (`pip install poetry`)
2. ✅ Run `poetry install` to install all package dependencies
3. ✅ Run database migrations (`python label_studio/manage.py migrate`)
4. ✅ Run collectstatic (`python label_studio/manage.py collectstatic`)
5. ✅ Start the server (`python label_studio/manage.py runserver`)

## Implementation Details

### Build Time vs Runtime

The Dockerfile separates concerns between build time and runtime:

**Build Time:**
- Install system dependencies
- Install Poetry
- Copy source code
- Install Python dependencies via `poetry install`

**Runtime (Container Startup):**
- Run database migrations
- Collect static files
- Start the Django development server

This approach ensures that migrations and static file collection happen with the latest code and database state when the container starts.

### Entrypoint Script

The entrypoint script (`/entrypoint.sh`) executes three commands in sequence:
1. Database migrations
2. Static file collection
3. Server startup

This ensures the database is always up-to-date and static files are properly collected before the server starts.

## File Structure

```
label-studio/
├── Dockerfile.server           # Main server-only Dockerfile
├── Dockerfile.server.md         # User-facing documentation
├── docker-compose.server.yml    # Docker Compose configuration
└── DOCKERFILE_SERVER_NOTES.md   # This file - implementation notes
```

## Known Limitations & Testing Notes

### Network/SSL Issues During Build

During development, network/SSL certificate verification issues were encountered in the build environment. These issues are environment-specific and typically occur in:
- Corporate networks with SSL inspection
- Networks with restrictive firewalls
- Environments with custom CA certificates

**Workarounds:**
1. Build with host network: `docker build --network=host -f Dockerfile.server -t label-studio-server:latest .`
2. Configure Docker daemon with appropriate DNS and proxy settings
3. Use the provided docker-compose.server.yml which may handle network issues better

### Production vs Development

This Dockerfile is designed for **development purposes only**. Key differences from the production Dockerfile:

| Feature | Dockerfile.server (Dev) | Dockerfile (Production) |
|---------|-------------------------|-------------------------|
| Web Server | Django runserver | Nginx + uWSGI |
| Frontend | Not built | Built and included |
| Multi-stage build | No | Yes |
| Security hardening | Basic | Advanced |
| Performance | Development-optimized | Production-optimized |

## Usage Examples

### Basic Usage
```bash
docker build -f Dockerfile.server -t label-studio-server:latest .
docker run -it -p 8080:8080 label-studio-server:latest
```

### With Persistent Data
```bash
docker run -it -p 8080:8080 \
  -v $(pwd)/label-studio-data:/label-studio/data \
  label-studio-server:latest
```

### Using Docker Compose
```bash
docker-compose -f docker-compose.server.yml up --build
```

## Verification Steps

To verify the Dockerfile works correctly:

1. **Build the image:**
   ```bash
   docker build -f Dockerfile.server -t label-studio-server:test .
   ```

2. **Run the container:**
   ```bash
   docker run -it -p 8080:8080 label-studio-server:test
   ```

3. **Check the logs for:**
   - "Running database migrations..." ✓
   - "Collecting static files..." ✓
   - "Starting Label Studio server at http://0.0.0.0:8080" ✓

4. **Access the server:**
   - Open browser to http://localhost:8080
   - Verify Label Studio UI loads

## Future Improvements

Potential improvements for future iterations:

1. **Multi-stage build** - Reduce final image size by using builder pattern
2. **Health checks** - Add Docker HEALTHCHECK instruction
3. **Non-root user** - Run as non-root user for better security
4. **Build caching** - Optimize layer caching for faster builds
5. **Configuration files** - Support custom Django settings via mounted config files

## Related Files

- `Dockerfile` - Production multi-stage Dockerfile with Nginx and frontend
- `Dockerfile.development` - Development Dockerfile used for testing
- `docker-compose.yml` - Full production stack with PostgreSQL and Nginx
- `README.md` - Main project documentation with setup instructions

## References

- Label Studio Documentation: https://labelstud.io/guide/
- Poetry Documentation: https://python-poetry.org/docs/
- Django Documentation: https://docs.djangoproject.com/
- Docker Best Practices: https://docs.docker.com/develop/dev-best-practices/
