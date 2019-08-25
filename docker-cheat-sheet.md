# Docker Cheat Sheet

## Build

```python
    docker build -t name_for_container .
```

## Run

```python
    docker-compose up
```

## Run and launch the command line

```python
    docker run -it name_for_container sh
```

## Stop all running containers

```python
    docker stop $(docker ps -aq)
```