# Docker Cheat Sheet

## Build container

```
    docker build -t name_for_container .
```

## Run

```
    docker-compose up
```

## Build all containers

```
    docker-compose build
```

## Run and launch the command line

```
    docker run -it name_for_container sh
```

## Stop all running containers

```
    docker stop $(docker ps -aq) #powershell or linux
    FOR /f "tokens=*" %i IN ('docker ps -q') DO docker stop %i #windows cmd
```

## List all containers

```
    docker container ls --all
```

## List all images

```
    docker image ls --all
```

## Save image to file

```
   docker save -o filepath/filename.tar repository:tag
```

## Load image from file

```
    docker load -i filepath/filename.tar
```



