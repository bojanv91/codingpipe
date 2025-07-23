---
title: "PostgreSQL development setup with Docker Compose"
date: 2023-08-09
dateUpdated: Last Modified
permalink: /posts/setting-up-postgresql-with-docker-compose/
tags:
  - Postgres
  - Docker
  - DevOps
layout: layouts/post.njk
---

I needed a shared PostgreSQL instance that's always available for demos, prototypes, and trial projects. Installing Postgres directly means dealing with Windows services, path configurations, and version management across system updates.

This Docker Compose setup gives me a persistent development database that's isolated from my system but accessible to any project. Perfect for when you want to quickly spin up a demo without setting up project-specific database infrastructure.

## Setup

Choose a permanent folder for your PostgreSQL setup (e.g., `D:\devops\docker\local_postgres` or `C:\docker\postgres`). Create a `docker-compose.yml` file there:

```yml
# docker-compose.yml
version: '3.1'
services:
  db:
    image: postgres
    ports:
      - "15432:5432"
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    volumes:
       - ./postgresql-data:/var/lib/postgresql/data
```

The `postgresql-data` folder will be created automatically in the same directory as your `docker-compose.yml` file when you first run the container.

Run with:

```bash
docker-compose up
```

Connect using:

```plaintext
Host: localhost
Port: 15432
Username: postgres
Password: postgres
```

## Why This Works

Port 15432 avoids system Postgres conflicts, `./postgresql-data` keeps data persistent, and `restart: always` survives reboots. One shared development database that's always available.
