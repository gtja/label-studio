# Dockerfile.server - Label Studio Server-Only Docker Image

## Overview

This Dockerfile builds a Label Studio server-only environment for development purposes. It follows the development setup instructions from the README.md:

1. Install poetry
2. Run `poetry install`
3. Run database migrations (`python label_studio/manage.py migrate`)
4. Run collectstatic (`python label_studio/manage.py collectstatic`)
5. Start the server with `python label_studio/manage.py runserver`

## Building the Image

To build the Docker image:

```bash
docker build -f Dockerfile.server -t label-studio-server:latest .
```

### Build Arguments

- `PYTHON_VERSION`: Python version to use (default: 3.12)

Example with custom Python version:

```bash
docker build -f Dockerfile.server --build-arg PYTHON_VERSION=3.11 -t label-studio-server:latest .
```

## Running the Container

### Basic Usage

```bash
docker run -it -p 8080:8080 label-studio-server:latest
```

The server will be available at `http://localhost:8080`.

### With Persistent Data

To persist data between container restarts, mount a volume:

```bash
docker run -it -p 8080:8080 \
  -v $(pwd)/label-studio-data:/label-studio/data \
  label-studio-server:latest
```

### With Environment Variables

You can pass environment variables to configure Label Studio:

```bash
docker run -it -p 8080:8080 \
  -e DJANGO_DB=default \
  -e POSTGRE_NAME=labelstudio \
  -e POSTGRE_USER=postgres \
  -e POSTGRE_PASSWORD=password \
  -e POSTGRE_PORT=5432 \
  -e POSTGRE_HOST=postgres \
  -v $(pwd)/label-studio-data:/label-studio/data \
  label-studio-server:latest
```

## Key Features

- **Development Mode**: Uses Django's `runserver` command for development
- **Automatic Migrations**: Runs database migrations on container startup
- **Static Files**: Collects static files automatically
- **SQLite by Default**: Uses SQLite database stored in `/label-studio/data`
- **Port 8080**: Server listens on port 8080

## Differences from Production Dockerfile

This server-only Dockerfile is designed for development and differs from the production `Dockerfile` in several ways:

1. **No Frontend Build**: Does not build frontend assets
2. **No Nginx**: Does not include Nginx reverse proxy
3. **Development Server**: Uses Django's `runserver` instead of uWSGI
4. **Simplified**: Focused on getting the server running quickly for development

## Environment Variables

- `PYTHONUNBUFFERED=1`: Ensures Python output is sent straight to terminal
- `PYTHONDONTWRITEBYTECODE=1`: Prevents Python from writing .pyc files
- `LABEL_STUDIO_BASE_DATA_DIR=/label-studio/data`: Data directory for Label Studio
- `DJANGO_SETTINGS_MODULE=core.settings.label_studio`: Django settings module

## Notes

- This image is intended for development and testing purposes
- For production deployments, use the main `Dockerfile` which includes Nginx and uWSGI
- The container runs migrations and collectstatic on every startup
- Data is stored in `/label-studio/data` by default (mount this as a volume for persistence)

## Troubleshooting

### Network Issues During Build

If you encounter SSL certificate errors or network connectivity issues during the `poetry install` step:

1. Try building with host network: `docker build --network=host -f Dockerfile.server -t label-studio-server:latest .`
2. Check your corporate proxy settings if behind a firewall
3. Try using a different DNS resolver in Docker daemon configuration

### Database Issues

If you see database-related errors:

1. Make sure the data directory is writable
2. Check that migrations complete successfully in the container logs
3. For PostgreSQL, ensure the database server is accessible and credentials are correct
