# docker_cheatsheet

# IMAGES

---

docker images                          → List local images
docker pull image:tag                  → Download image from registry
docker build -t name:tag .             → Build image from Dockerfile
docker build --no-cache -t name:tag .  → Build without cache
docker rmi image_id                    → Remove image
docker rmi $(docker images -q)         → Remove all images
docker tag old_name new_name:tag       → Rename/tag an image
docker image prune                     → Remove unused images

---

## CONTAINERS

docker ps                              → List running containers
docker ps -a                           → List all containers (inc. stopped)
docker run image                       → Run container (foreground)
docker run -d image                    → Run detached (background)
docker run -it image bash              → Run interactive shell
docker run --name myapp -d image       → Run with custom name
docker run -p 8080:80 image            → Map host:container ports
docker run -v /host/path:/container    → Mount volume
docker run --rm image                  → Auto-remove on exit
docker run --env-file .env image       → Load env vars from file
docker stop container_id               → Graceful stop
docker kill container_id               → Force stop
docker start container_id              → Start stopped container
docker restart container_id            → Restart container
docker rm container_id                 → Remove stopped container
docker rm -f container_id              → Force remove running container
docker rm $(docker ps -aq)             → Remove all stopped containers

---

## INTERACTING WITH CONTAINERS

docker exec -it container_id bash      → Shell into running container
docker exec container_id command       → Run command in container
docker logs container_id               → View logs
docker logs -f container_id            → Follow (tail) logs
docker logs --tail 100 container_id    → Last 100 lines
docker cp file.txt container_id:/path  → Copy file into container
docker cp container_id:/path file.txt  → Copy file out of container
docker inspect container_id            → Full container metadata (JSON)
docker stats                           → Live CPU/mem usage
docker top container_id                → Running processes inside container

---

## VOLUMES

docker volume ls                       → List volumes
docker volume create my_vol            → Create named volume
docker volume inspect my_vol           → Volume details
docker volume rm my_vol                → Remove volume
docker volume prune                    → Remove unused volumes
docker run -v my_vol:/app/data image   → Attach named volume

---

## NETWORKS

docker network ls                      → List networks
docker network create my_net           → Create custom network
docker network inspect my_net          → Network details
docker network rm my_net               → Remove network
docker run --network my_net image      → Run on custom network
docker network connect my_net container_id   → Connect container to network
docker network disconnect my_net container_id → Disconnect

---

## DOCKER COMPOSE

docker compose up                      → Start services (foreground)
docker compose up -d                   → Start detached
docker compose down                    → Stop & remove containers
docker compose down -v                 → Also remove volumes
docker compose build                   → Build/rebuild images
docker compose ps                      → List compose services
docker compose logs -f                 → Follow all service logs
docker compose logs -f service_name    → Follow specific service
docker compose exec service bash       → Shell into service
docker compose restart service         → Restart one service
docker compose pull                    → Pull latest images

---

## CLEANUP

docker system prune                    → Remove all unused resources
docker system prune -a                 → Remove everything unused (inc. images)
docker system df                       → Show disk usage

---

## DOCKERFILE REFERENCE

FROM ubuntu:22.04           → Base image
WORKDIR /app                → Set working directory
COPY . .                    → Copy files from host
RUN apt-get update          → Run command during build
ENV PORT=8080               → Set environment variable
EXPOSE 8080                 → Document port (informational)
CMD ["python", "[app.py](http://app.py/)"]    → Default run command
ENTRYPOINT ["python"]       → Fixed entrypoint
ARG VERSION=1.0             → Build-time variable