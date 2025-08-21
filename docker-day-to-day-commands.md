```bash
docker compose -f ops/docker-compose.yml ps                  # see running containers in this compose
docker compose -f ops/docker-compose.yml logs -f              # stream logs
docker compose -f ops/docker-compose.yml exec rabbitmq rabbitmq-diagnostics ping    # exec into a container (e.g., check broker)
docker compose -f ops/docker-compose.yml restart rabbitmq  # restart just RabbitMQ
docker compose -f ops/docker-compose.yml down              # stop containers (keeps volumes)
docker compose -f ops/docker-compose.yml down -v            # stop and remove volumes (wipes data)
```


# Docker CLI (Most-Used)

## Basics

```bash
docker --version
docker info
docker help
```

## Images

```bash
docker search nginx
docker pull nginx:latest
docker images                        # list local images
docker image inspect nginx:latest
docker rmi IMAGE_ID                  # remove image
docker image prune                   # remove dangling
docker build -t myapp:1.0 .          # build from Dockerfile in .
docker build -t myapp:1.0 -f Dockerfile.prod .
docker tag myapp:1.0 myrepo/myapp:1.0
docker login
docker push myrepo/myapp:1.0
docker save -o myimg.tar myapp:1.0   # export image to tar
docker load -i myimg.tar             # import image from tar
```

## Containers (run/start/stop)

```bash
# Run (detached) with name, port, env, volume, restart policy
docker run -d --name web \
  -p 8080:80 -e ASPNETCORE_ENVIRONMENT=Production \
  -v mydata:/var/lib/app --restart unless-stopped \
  myrepo/myapp:1.0

docker ps                            # running
docker ps -a                         # all
docker stop CONTAINER
docker start CONTAINER
docker restart CONTAINER
docker rm CONTAINER
docker container prune               # remove stopped
docker kill CONTAINER                # SIGKILL
docker wait CONTAINER                # wait for exit, print status code

# One-off command, auto-remove, interactive shell
docker run --rm -it alpine:3.20 sh
```

## Inspect, Logs, Exec, Stats

```bash
docker logs CONTAINER
docker logs -f --tail 100 CONTAINER
docker exec -it CONTAINER bash       # or sh
docker top CONTAINER                 # processes
docker stats                         # live CPU/mem/IO
docker inspect CONTAINER_OR_IMAGE
docker port CONTAINER                # show port mappings
docker events                        # real-time daemon events
```

## Copying Files

```bash
docker cp CONTAINER:/path/in/container ./local/dir
docker cp ./local/file CONTAINER:/path/in/container
```

## Volumes

```bash
docker volume create mydata
docker volume ls
docker volume inspect mydata
docker volume rm mydata
docker volume prune
# Use in run:
docker run -v mydata:/var/lib/app myrepo/myapp:1.0
# Bind mount (host path -> container path):
docker run -v $(pwd)/config:/app/config:ro myrepo/myapp:1.0
```

## Networks

```bash
docker network ls
docker network create --driver bridge mynet
docker network inspect mynet
docker network connect mynet CONTAINER
docker network disconnect mynet CONTAINER
docker network rm mynet
docker network prune
# Use on run:
docker run --network mynet --name api myrepo/myapp:1.0
```

## Resource Limits (handy defaults)

```bash
docker run -d --name busy \
  --cpus="1.5" --memory="512m" --memory-swap="512m" \
  alpine sh -c "while true; do :; done"
```

## Multi-arch / Platform

```bash
docker run --platform linux/amd64 myrepo/myapp:1.0
```

## System Housekeeping

```bash
docker system df                     # disk usage
docker system prune                  # remove unused (prompt)
docker system prune -a               # remove unused + images
docker builder prune                 # prune build cache
```

## Docker Compose (v2: `docker compose …`)

```bash
docker compose version
docker compose up -d                 # start (build if needed)
docker compose up -d --build         # force rebuild
docker compose ps
docker compose logs -f SERVICE
docker compose exec SERVICE sh       # or bash
docker compose stop
docker compose start
docker compose restart SERVICE
docker compose down                  # stop & remove
docker compose down -v               # also remove named volumes
docker compose pull                  # pull images
docker compose build                 # build services
docker compose run --rm SERVICE cmd  # one-off task
```

## BuildKit & Buildx (faster builds, multi-arch)

```bash
# Enable BuildKit (per command)
DOCKER_BUILDKIT=1 docker build -t myapp:dev .

# Buildx basics
docker buildx ls
docker buildx create --name multi && docker buildx use multi
docker buildx build --platform linux/amd64,linux/arm64 \
  -t myrepo/myapp:1.0 --push .
```

## Contexts (switch Docker endpoints)

```bash
docker context ls
docker context create myctx --docker "host=ssh://user@host"
docker context use myctx
docker context use default
```

## Quick Troubleshooting

```bash
docker logs -f CONTAINER
docker inspect CONTAINER | jq '.[0].State'   # status/timestamps
docker exec -it CONTAINER sh
docker events --since 10m
docker system df
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Status}}\t{{.Names}}"
```

## Handy Patterns

```bash
# Health check (in Dockerfile)
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# Pass env file to container
docker run --env-file .env -d myrepo/myapp:1.0

# Name+tag from git commit
docker build -t myapp:$(git rev-parse --short HEAD) .

# Clean everything unused (dangling + stopped + networks + cache)
docker system prune -a --volumes
```
