# trouble shoot:

1. Below error happens when you run docker in a VM or AWS environment, you need to restart docker service
  [docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.](https://forums.docker.com/t/cannot-connect-to-the-docker-daemon-at-unix-var-run-docker-sock-is-the-docker-daemon-running/32818/3)
  ```
  sudo service docker stop
  sudo service docker start
  ```

2. docker config
  [image location change](https://forums.docker.com/t/how-do-i-change-the-docker-image-installation-directory/1169)
  CAUTION: on CentOS 7, don't follow this.

  You can change Docker's storage base directory (where container and images go) using the -g option when starting the Docker daemon.

  Ubuntu/Debian: edit your /etc/default/docker file with the -g option: DOCKER_OPTS="-dns 8.8.8.8 -dns 8.8.4.4 -g /mnt"

  Fedora/Centos: edit /etc/sysconfig/docker, and add the -g option in the other_args variable: ex. other_args="-g /var/lib/testdir". If there's more than one option, make sure you enclose them in " ". After a restart, (service docker restart) Docker should use the new directory.

  Using a symlink is another method to change image storage.

3. can't load image `docker pull python` python is an available image in Dockerhub  
  [question](https://stackoverflow.com/questions/23111631/cannot-download-docker-images-behind-a-proxy)   
  [HTTP/HTTPS proxy doc](https://docs.docker.com/engine/admin/systemd/#runtime-directory-and-storage-driver)   
  
  A quick outline of using systemctl :
  First, create a systemd drop-in directory for the docker service:  
  ```
  sudo mkdir -p /etc/systemd/system/docker.service.d
  ```  
  Now create a file called `/etc/systemd/system/docker.service.d/http-proxy.conf` that adds the HTTP_PROXY environment         variable:
  ```
  [Service]
  Environment="HTTP_PROXY=http://proxy.example.com:80/"
  ```
  and/or create a file called `/etc/systemd/system/docker.service.d/https-proxy.conf` that adds the HTTPS_PROXY environment  
  If you have internal Docker registries that you need to contact without proxying you can specify them via the               NO_PROXY environment variable:
  ```
  Environment="HTTP_PROXY=http://proxy.example.com:80/"
  Environment="NO_PROXY=localhost,127.0.0.0/8,docker-registry.somecorporation.com"
  ```
  Flush changes:
  `$ sudo systemctl daemon-reload`
  Verify that the configuration has been loaded:
  ```
  $ sudo systemctl show --property Environment docker
  Environment=HTTP_PROXY=http://proxy.example.com:80/
  ```
  Restart Docker:
  `$ sudo systemctl restart docker`

4. [How to mount host directory in docker container?](https://stackoverflow.com/questions/23439126/how-to-mount-host-directory-in-docker-container)   
to mount the the current directory (source) with /test_container (target) we are going to use:
```
  docker run -it --mount src="$(pwd)",target=/test_container,type=bind k3_s3
```

5. Dockerfile to create your own image
  ```
  $ FROM python COPY . /src CMD [“python”, “/src/birthday.py”]
  FROM requires either one or three arguments
  ```

# Simple example using Docker to run Python  
https://www.civisanalytics.com/blog/using-docker-to-run-python/  
change the Dockerfile based on the help https://codefresh.io/docker-guides/build-docker-image-dockerfiles/
Dockerfile
a file without extension
```
FROM python 
ADD "./Birthday-Problem/birthday.py" "./src/birthday.py"
CMD ["python", "/src/birthday.py"]
```
To build from current dir ./ and specify a name
```
$ sudo docker build ./ -t ydd9/python-birthday
Successfully built ydd9/python-birthday:latest
$ sudo docker build ./
Successfully built da3bf65a40da
```

To run the app in docker, just use command `docker run <built image>`  
```
sudo docker run ydd9/python-birthday
# but below works as well
sudo docker run ydd9/python-birthday python /src/birthday.py
```

# minikube install
minikube will start a Kubernetes cluster on your local or VM environment and you can use it for learning kubernetes commands.   
Install minikube https://github.com/kubernetes/minikube#using-rkt-container-engine
```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /opt/bin/

curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl && chmod +x kubectl
```

https://stackoverflow.com/questions/37810293/coreos-read-only-file-system  
In CoreOS the /usr partition is read-only by design, so /usr/local/bin/ will be read-only too.  use /opt/ for this purpose. 
Do `mkdir -p /opt/bin; mv ./kubectl /opt/bin/kubectl`, minikube still can't run as it always goes to /usr path to write.


# kubeadm install
kubeadm is designed to help you create Kubernetes cluster env on VMs or cloud hosted systems. It provides the minimum setup of Kubernetes cluster. Now I'm using Debian9.3 stretch in VirtualBox5.2

First, install Docker-CE in Debian9.3 stretch if it is not installed. You can check by `sudo docker run hello-world`
Follow official installation and check the part for Debian stretch version, the link is     https://docs.docker.com/engine/installation/linux/docker-ce/debian/#set-up-the-repository       
Pay attention to the Docker version, do not install latest Docker-CE which Kubernetes does not support, install Docker-CE <= 17.03 for now (Dec 2017)    

Then install kubeadm https://kubernetes.io/docs/setup/independent/install-kubeadm/ 
```
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```

On your desired Kubernetes cluster master, you can init a cluster   
In case of error, go to check solution https://github.com/kubernetes/kubernetes/issues/53333  
```
 [ERROR Swap]: running with swap on is not supported. Please disable swap
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-p
```

try solution one or two:
```
# solution one
iptables -F
swapoff -a
free -m
kubeadm reset

kubeadm init
```

```
# solution two
# delete the nodes if you have.
kubeadm reset
# add "Environment="KUBELET_EXTRA_ARGS=--fail-swap-on=false"" to /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl daemon-reload
systemctl restart kubelet

kubeadm init
```

Once init successfully finish, copy the message and you will need to use it to add other nodes into this cluster.
```
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token cf016f.b7fd1ad86da928cf 10.0.2.15:6443 --discovery-token-ca-cert-hash sha256:e7bd6e7561fda671554625787e8a71be7818a4f071d91b9bf78ec815556b8023
```

Fail to add node into Kubernetes cluster
In another VM login as root and add it into cluster
```
$ kubeadm join --token cf016f.b7fd1ad86da928cf 10.0.2.15:6443 --discovery-token-ca-cert-hash sha256:e7bd6e7561fda671554625787e8a71be7818a4f071d91b9bf78ec815556b8023

Failed to request cluster info, will try again: [Get https://10.0.2.15:6443/api/v1/namespaces/kube-public/configmaps/cluster-info: dial tcp 10.0.2.15:6443: getsockopt: connection refused]
```


