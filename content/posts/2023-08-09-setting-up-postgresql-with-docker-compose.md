---
title: "My PostgreSQL development setup with Docker Compose"
date: 2023-08-09
dateUpdated: Last Modified
permalink: /posts/setting-up-postgresql-with-docker-compose/
tags:
  - Postgres
  - Docker
layout: layouts/post.njk
---

Docker is a great tool for simplifying the installation of applications and services. As well as isolating them from your machine to avoid potential conflicts with other installations. This is especially useful for development machines where you tend to have many services installed in various versions for various projects.

In this post, we'll install and setup Postgres with Docker Desktop, and we'll configure data persistence in the Docker container.

Prerequisite for this post is to have [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed.

First, start Docker Desktop.

Next, create or choose a folder where your Docker container setup files will be stored. My folder of choice is `D:\devops\docker\local_postgres`.

Next, inside your chosen folder, create a `docker-compose.yml` file and place the following content:

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

The `docker-compose.yml` file will download and install the Postgres image, will set the DB engine with the provided environment variables, will bind the internal port `5432` to an external port `15432`, and finally will configure an external volume where the Postgres data files be stored and persistent outside of the container. The internal `/var/lib/postgresql/data` folder will be mapped to our 
`postgres-data` folder on the disk (or the full path: `D:\devops\docker\local_postgres\postgres-data`).

Next, open the Windows terminal in your folder and run the following command:

```
docker-compose up
```

The output will show that Docker is working correctly. You can also inspect the status of your container instance in the Docker Desktop app as well.

Now you can connect to the container's Postgres database engine using your DB tool of choice (e.g. DBeaver) with the following connection info:

```
Host: localhost
Port: 15432
Username: postgres
Password: postgres
```

Stopping and restarting the docker container won't cause data loss due the presence of the volume parameter in docker-compose.

