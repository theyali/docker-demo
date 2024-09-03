# Docker

#### Сбилдить образ
```
docker build . --file Dockerfile --tag image-tag
```

Dockerfile используется по умолчанию, указывать его нужно только если используется другое имя файла

#### Запустить образ

```
docker run image_id/image_tag
```

#### Открыть консоль контейнера

```
docker exec -it container_id /bin/sh
```

#### Вывести список образов

```
docker images
docker image -ls 
```

#### Вывести список запущенных контейнеров

```
docker ps
docker ps -a  # все, включая остановленные
```

#### Удаление

+ Удалить image:

```
docker rmi image_id
```

+ Удалить container:

```
docker rm container_id
```

+ Удалить все остановленные контейнеры, networks, образы (которые не связаны хотя бы с одним контейнером)

```
docker system prune -a
```



# Django Application Deployment with Docker, Docker Compose, and Nginx

This guide will help you deploy a Django application using Docker, Docker Compose, and Nginx on a fresh server.

## Prerequisites

Before you begin, ensure you have the following:

- A fresh server with a Linux distribution (e.g., Ubuntu 20.04)
- SSH access to the server
- A basic understanding of Docker and Docker Compose

## Step 1: Install Docker

1. Update your existing list of packages:

    ```bash
    sudo apt-get update
    ```

2. Install Docker using the convenience script:

    ```bash
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    ```

3. Verify that Docker is installed correctly:

    ```bash
    sudo docker --version
    ```

## Step 2: Install Docker Compose

1. Download the latest version of Docker Compose:

    ```bash
    sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep -Po '"tag_name": "\K.*\d')/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    ```

2. Apply executable permissions to the binary:

    ```bash
    sudo chmod +x /usr/local/bin/docker-compose
    ```

3. Verify the installation:

    ```bash
    docker-compose --version
    ```

## Step 3: Set Up Your Project

1. Clone your project repository:

    ```bash
    git clone https://github.com/yourusername/yourproject.git
    cd yourproject
    ```

2. Set up your `.env` file with the necessary environment variables:

    ```plaintext
    SECRET_KEY='your-secret-key'
    DEBUG=True
    DATABASE_URL=postgres://user:password@db:5432/yourdatabase
    ```

3. Ensure your `docker-compose.yml` and `Dockerfile` are correctly configured to build and run your Django application.

### Example `docker-compose.yml`:

```yaml
version: '3.9'

services:
  web:
    build: .
    command: sh -c "python manage.py migrate && daphne -b 0.0.0.0 -p 8000 yourproject.asgi:application"
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - SECRET_KEY=${SECRET_KEY}
      - DEBUG=${DEBUG}
    env_file:
      - .env

  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - POSTGRES_DB=yourdatabase
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    ports:
      - "5432:5432"

  redis:
    image: redis:6
    ports:
      - "6379:6379"

volumes:
  postgres_data:
