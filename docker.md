# Docker Cheat Sheet

## Images
```bash
docker images             # list local images
docker pull nginx:latest
docker build -t myapp:1.0 .
docker build --no-cache -t myapp:1.0 .
docker rmi myapp:1.0      # remove image
docker image prune -a     # remove all unused images
docker tag myapp:1.0 myrepo/myapp:1.0
docker push myrepo/myapp:1.0
docker history myapp:1.0  # show layer history
```

## Containers
```bash
docker run -it ubuntu bash                    # interactive shell
docker run -d --name web -p 8080:80 nginx     # detached, port map, named
docker run --rm -it alpine sh                 # auto-remove on exit
docker run -e KEY=value -e KEY2=value2 myapp  # env vars
docker run --env-file .env myapp
docker run -v $(pwd):/app myapp               # bind mount
docker run -v mydata:/var/lib/data myapp      # named volume
docker ps                                     # running containers
docker ps -a                                  # all containers (incl. stopped)
docker stop web && docker start web
docker restart web
docker rm web                                 # remove stopped container
docker rm -f web                              # force remove running container
```

## Interacting with running containers
```bash
docker exec -it web bash       # shell into running container
docker logs web                # view logs
docker logs -f web             # follow logs
docker logs --tail 100 -f web  # follow last 100 lines
docker stats                   # live resource usage
docker top web                 # processes in container
docker cp web:/app/file.txt .  # copy file out of container
docker cp file.txt web:/app/   # copy file into container
docker inspect web             # full metadata (JSON)
docker inspect -f '{{.NetworkSettings.IPAddress}}' web
```

## Cleanup
```bash
docker system df                  # disk usage summary
docker system prune               # remove stopped containers, dangling images, unused networks
docker system prune -a --volumes  # aggressive cleanup (careful: removes volumes too)
docker container prune            # remove all stopped containers
docker volume prune               # remove unused volumes
docker network prune              # remove unused networks
```

## Docker Compose
```bash
docker compose up -d       # start in background
docker compose up --build  # rebuild then start
docker compose down        # stop and remove containers
docker compose down -v     # also remove volumes
docker compose logs -f service_name
docker compose ps
docker compose exec service_name bash
docker compose restart service_name
docker compose config      # validate/print resolved config
```

## Networking & volumes
```bash
docker network ls
docker network create mynet
docker network connect mynet web
docker volume ls
docker volume create mydata
docker volume inspect mydata
```

## Useful one-liners
```bash
docker run --rm -it --entrypoint bash myimage      # override entrypoint to debug
docker exec -it $(docker ps -q -f name=web) bash   # exec by name filter
docker ps -q | xargs docker stop                   # stop all running containers
docker rmi $(docker images -f "dangling=true" -q)  # remove dangling images
docker inspect --format='{{.State.ExitCode}}' web  # get exit code
docker events                                      # stream Docker daemon events
docker save myapp:1.0 -o myapp.tar                 # export image to tarball
docker load -i myapp.tar                           # import image from tarball
```
