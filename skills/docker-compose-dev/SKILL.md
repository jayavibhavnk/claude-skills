---
name: docker-compose-dev
description: Set up local development environments with Docker Compose - multi-service apps, databases, and debugging.
metadata:
  priority: 8
  docs:
    - "https://docs.docker.com/compose/"
  pathPatterns:
    - "docker-compose*.yml"
    - "docker-compose*.yaml"
  bashPatterns:
    - '\bdocker-compose\b'
    - '\bdocker\b'
  promptSignals:
    phrases:
      - "docker"
      - "container"
      - "docker-compose"
    anyOf:
      - "dev environment"
      - "containerize"
      - "local dev"
---

## Docker Compose for Development

### Basic Structure

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgres://postgres:secret@db:5432/myapp
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Development vs Production

```yaml
# docker-compose.yml (base)
version: '3.8'

services:
  app:
    build:
      context: .
      target: ${BUILD_TARGET:-development}
    # ...

# docker-compose.yml (development)
services:
  app:
    environment:
      - NODE_ENV=development
    volumes:
      - .:/app
      - /app/node_modules
    command: npm run dev

# docker-compose.yml (production)
services:
  app:
    environment:
      - NODE_ENV=production
    command: npm start
    restart: unless-stopped
```

### Multi-Service App

```yaml
services:
  app:
    build: .
    depends_on:
      api:
        condition: service_healthy
      queue:
        condition: service_started

  api:
    build: ./services/api
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  worker:
    build: ./services/worker
    depends_on:
      - queue
      - db

  queue:
    image: rabbitmq:3-management

  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
```

### Debugging in Containers

```yaml
services:
  app:
    build: .
    # Enable debugger
    command: ["node", "--inspect=0.0.0.0:9229", "server.js"]
    ports:
      - "9229:9229"  # Debug port
      - "3000:3000"  # App port
    environment:
      - NODE_OPTIONS="--inspect"
```

### Common Commands

```bash
# Start all services
docker-compose up

# Start in background
docker-compose up -d

# Rebuild after changes
docker-compose up --build

# View logs
docker-compose logs -f app

# Run one-off command
docker-compose run --rm app npm test

# Stop and remove
docker-compose down

# Stop and remove volumes
docker-compose down -v

# Scale service
docker-compose up -d --scale worker=3
```

### Database Services

```yaml
# PostgreSQL with init script
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
      - postgres_data:/var/lib/postgresql/data

# MySQL
services:
  mysql:
    image: mysql:8
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: myapp
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

# MongoDB
services:
  mongo:
    image: mongo:7
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secret
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
```

### Best Practices

1. Use specific image tags, not `latest`
2. Use named volumes for persistent data
3. Use `depends_on` with `condition: service_healthy`
4. Set resource limits in production
5. Use `.dockerignore` to exclude files
6. Use multi-stage builds for smaller images
7. Never store secrets in docker-compose files
8. Use healthchecks for dependency readiness

### Useful Patterns

```yaml
# Wait for service to be ready
services:
  app:
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```
