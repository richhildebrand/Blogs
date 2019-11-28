## Installation 

Several things must be installed to spin up your first kubernetes cluster.

To install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux), run the following command.

```
    sudo apt-get update && sudo apt-get install -y apt-transport-https
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
    sudo apt-get update
    sudo apt-get install -y kubectl
```

To install [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/), run the following commands.

```
    curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
    && chmod +x minikube
    sudo mkdir -p /usr/local/bin/
    sudo install minikube /usr/local/bin/
```

To install [docker](https://www.docker.com/)
=80
```
    sudo apt install docker.io
    sudo systemctl start docker
    sudo systemctl enable docker
```

## Start minikube and load our image

For this demo, I am using an Angular2 application that runs on port 4200, so that is the port we will expose. A `.tar file` can be created from a docker image by running `docker save -o c:/myfile.tar image-name`.

```
    sudo minikube start --vm-driver=none
    sudo docker load -i filepath/filename.tar
    sudo kubectl run kube-job-name --image=docker-image-name --port=4200 --image-pull-policy=IfNotPresent
    sudo kubectl expose deployment kube-job-name --type=NodePort --port=4200
```

## Confirm we are up and running

You can see all running pods along with their urls by running 

```
    sudo kubectl get pods
    sudo minikube service kube-job-name --url
```

## Notes

If you are receiving the `remote error: tls: bad certificate` error. This can be fixed by running the command `minikube delete` and starting again from `sudo minikube start --vm-driver=none`