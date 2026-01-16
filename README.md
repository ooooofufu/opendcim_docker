# openDCIM (Docker Compose)

## Prerequisites, check version
- Docker
docker --version

- Docker Compose
docker compose version

## Quick start
1) Copy env template and edit values:
   cp .env.example .env
   nano .env

2) Start:
   docker compose up -d

3) Open:
   http://localhost:${DCIM_HTTP_PORT:-8081}/index.php

## Stop
docker compose down

## Logs
docker compose logs --tail=200 opendcim
docker compose logs --tail=200 db
