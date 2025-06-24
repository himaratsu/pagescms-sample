---
layout: default
title: Configuration
nav_order: 4
description: "Setup and configuration instructions"
---

# Configuration Guide
{: .no_toc }

Learn how to configure and customize your application for different environments.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Environment Configuration

### Development Environment

Create a `.env.development` file in your project root:

```bash
# API Configuration
API_URL=http://localhost:8080/api/v1
API_KEY=dev_api_key_123
DEBUG=true

# Database Configuration
DATABASE_URL=postgresql://user:password@localhost:5432/myapp_dev
REDIS_URL=redis://localhost:6379

# Feature Flags
ENABLE_ANALYTICS=false
ENABLE_CACHING=true
ENABLE_LOGGING=true
```

### Production Environment

For production, use these recommended settings:

```bash
# API Configuration
API_URL=https://api.yourapp.com/v1
API_KEY=prod_api_key_secure_123
DEBUG=false

# Database Configuration
DATABASE_URL=postgresql://user:password@prod-db:5432/myapp_prod
REDIS_URL=redis://prod-redis:6379

# Feature Flags
ENABLE_ANALYTICS=true
ENABLE_CACHING=true
ENABLE_LOGGING=true

# Security
SESSION_SECRET=your_super_secure_session_secret
JWT_SECRET=your_jwt_secret_key
CORS_ORIGIN=https://yourapp.com
```

## Application Configuration

### Basic Settings

Configure your application using the `config.json` file:

```json
{
  "app": {
    "name": "My Application",
    "version": "1.0.0",
    "port": 3000,
    "host": "localhost"
  },
  "database": {
    "type": "postgresql",
    "host": "localhost",
    "port": 5432,
    "database": "myapp",
    "username": "user",
    "password": "password",
    "ssl": false,
    "pool": {
      "min": 2,
      "max": 10
    }
  },
  "cache": {
    "type": "redis",
    "host": "localhost",
    "port": 6379,
    "ttl": 3600
  }
}
```

### Advanced Settings

For more advanced configurations, use the `config.advanced.json`:

```json
{
  "logging": {
    "level": "info",
    "format": "json",
    "file": "./logs/app.log",
    "maxFiles": 5,
    "maxSize": "10m"
  },
  "security": {
    "bcrypt": {
      "rounds": 12
    },
    "jwt": {
      "expiresIn": "7d",
      "algorithm": "HS256"
    },
    "cors": {
      "origin": ["http://localhost:3000"],
      "credentials": true
    }
  },
  "features": {
    "rateLimit": {
      "windowMs": 900000,
      "max": 100
    },
    "upload": {
      "maxFileSize": "10mb",
      "allowedTypes": ["image/jpeg", "image/png", "application/pdf"]
    }
  }
}
```

## Database Configuration

### PostgreSQL Setup

1. Install PostgreSQL:
   ```bash
   # Ubuntu/Debian
   sudo apt-get install postgresql postgresql-contrib
   
   # macOS with Homebrew
   brew install postgresql
   ```

2. Create database and user:
   ```sql
   CREATE DATABASE myapp_dev;
   CREATE USER myapp_user WITH PASSWORD 'secure_password';
   GRANT ALL PRIVILEGES ON DATABASE myapp_dev TO myapp_user;
   ```

3. Update your environment variables:
   ```bash
   DATABASE_URL=postgresql://myapp_user:secure_password@localhost:5432/myapp_dev
   ```

### Redis Setup

1. Install Redis:
   ```bash
   # Ubuntu/Debian
   sudo apt-get install redis-server
   
   # macOS with Homebrew
   brew install redis
   ```

2. Start Redis service:
   ```bash
   # Ubuntu/Debian
   sudo systemctl start redis-server
   
   # macOS
   brew services start redis
   ```

## Web Server Configuration

### Nginx Configuration

Example Nginx configuration for production:

```nginx
server {
    listen 80;
    server_name yourapp.com www.yourapp.com;
    
    # Redirect HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourapp.com www.yourapp.com;
    
    # SSL Configuration
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    
    # Security Headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    
    # Proxy to Node.js application
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
    
    # Static file serving
    location /static/ {
        alias /path/to/static/files/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

## Monitoring Configuration

### Application Monitoring

Configure application monitoring using environment variables:

```bash
# Monitoring
MONITORING_ENABLED=true
METRICS_PORT=9090
HEALTH_CHECK_ENDPOINT=/health

# APM (Application Performance Monitoring)
APM_SERVICE_NAME=myapp
APM_SERVER_URL=https://apm.yourapp.com
APM_SECRET_TOKEN=your_apm_token
```

### Health Check Configuration

```json
{
  "healthCheck": {
    "timeout": 5000,
    "checks": [
      {
        "name": "database",
        "type": "postgresql",
        "critical": true
      },
      {
        "name": "redis",
        "type": "redis",
        "critical": false
      },
      {
        "name": "external-api",
        "type": "http",
        "url": "https://api.external.com/health",
        "critical": false
      }
    ]
  }
}
```

## Troubleshooting Configuration

### Common Issues

1. **Port already in use**: Change the port in your configuration
2. **Database connection failed**: Check your DATABASE_URL
3. **Redis connection failed**: Ensure Redis is running
4. **SSL certificate errors**: Verify certificate paths and permissions

### Debug Mode

Enable debug mode for detailed logging:

```bash
DEBUG=true
LOG_LEVEL=debug
```

This will provide detailed information about configuration loading and application startup.