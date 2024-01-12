
<div align="center">
  <h3 align="center">Devops Engineering - Bare Metal Kubernetes</h3>

  <p align="center">
    A setup for a kubernetes cluster with deploying airbyte.
    <br />
    <strong>Explore the docs »</strong>
    <br />
    <br />
    
  </p>
</div>


<details id="#contents">
  <summary>Table of Contents</summary>
  <ol>
   <ul>
        <li><a href="#prerequisites">Common kubernetes setup for all nodes</a></li>
        <li><a href="#installation">Configuration for Master node and creating the cluster</a></li>
        <li><a href="#prerequisites">Joining worker nodes to the cluster</a></li>
        <li><a href="#installation">Configuring helm</a></li>
        <li><a href="#installation">Deploying Airbyte</a></li>
   </ul>
  </ol>
</details>

<h2>Common kubernetes setup for all nodes</h2>
<strong>Disable Swap</strong>

You might know about swap space on hard drives, which OS systems try to use as if it were RAM. Operating systems try to move less frequently accessed data to the swap space to free up RAM for more immediate tasks. However, accessing data in swap is much slower than accessing data in RAM because hard drives are slower than RAM.

Kubernetes schedules work based on the understanding of available resources. If workloads start using swap, it can become difficult for Kubernetes to make accurate scheduling decisions. Therefore, it’s recommended to disable swap before installing Kubernetes.

You can use the following comands: 

```sh
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

<strong>Set up hostnames</strong>

In this section we will change the hostnames for our virtual machines for easy navigation when using terminal.
In our case, I had to change the name of one virtual machine in order to have the following names: 
master
worker1
worker2
The command used is: 


```sh
sudo hostnamectl set-hostname master
sudo reboot
```

<strong>Install docker</strong>


1- Uninstall old versions

Older versions of Docker went by docker or docker-engine. Uninstall any such older versions before attempting to install a new version, along with associated dependencies. Also uninstall Podman and the associated dependencies if installed already:

```sh
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine \
                  podman \
                  runc
```
2- Edit the repository file for docker-ce manually since it is not available for rhel distributions. We have to recover it from centos servers.

```sh
sudo nano /etc/yum.repos.d/docker-ce.repo
```

change the docker-ce-stable section as follows:

```txt
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://download.docker.com/linux/centos/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://download.docker.com/linux/centos/gpg
```

3- Set up the repository for other docker related packages:

Install the yum-utils package (which provides the yum-config-manager utility) and set up the repository.

```sh
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```
4- Install Docker Engine
```sh
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```
5- Start Docker and enable it on startup:

```sh
sudo systemctl start docker
sudo systemctl enable docker
```

6- Check docker version:
```sh
sudo docker version
```
Now since we have docker up and running on all the virtual Machines, we need to configure containerd which will be the container runtime engine for our kubernetes cluster. 
