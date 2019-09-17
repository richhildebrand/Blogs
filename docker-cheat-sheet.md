# Docker Cheat Sheet

## Build container

```python
    docker build -t name_for_container .
```

## Run

```python
    docker-compose up
```

## Build all containers

```python
    docker-compose build
```

## Run and launch the command line

```python
    docker run -it name_for_container sh
```

## Stop all running containers

```python
    docker stop $(docker ps -aq)
```

## List all containers

```python
    docker container ls --all
```