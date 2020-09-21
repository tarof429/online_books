# Part 1: Introduction & Setup

## Introduction

ArchLinux is my favorite Linux distribution, and in this tutorial I am going to explain how to develop, deploy and manage a Kubernetes clusters on ArchLinux. We will first setup our development environment to the point where we can deploy pods to Minikube, a single node Kubernetes cluster. Next, we will take an exisiting Go application, turn it into a microservice, and deploy it to Minikube. Then to make our deployment more realistic, we will deploy some services and start to explore some tools to get insights into our cluster. In the next step, will will create a CI/CD pipeline and deploy our application to the cloud. We round out our tutorial with an exploration into additional tools used to deploy Kubernetes clusters. Finally, we will attempt to come up with a concise list of tools that you need to successfully use Kubernetes with ArchLinux, and delve into discussions into

The technology stack tht we'll look into is as follows:

* ArchLinux
* Docker
* Visual Studio Code
* Minikube
* Kubernetes
* Ansible
* Go
* Python
* Jenkins
* Spinaker
* GKE
* AWS
* Github
* Gitbook
* Prometheus

## How to view this book

Although this book is hosted on Github, it is optimized to be accessed using Gitbook at https://tarof429.gitbook.io/online-books/kubernetes_with_archlinux. 

## Setting up your development environment

=== Configuring Docker If you've been using Docker on your machine for any amount of time, chances are you have lots of containers and images that are eating valuable disk space. If possible, take this opportunity to clean up your system! You can use `docker system prune` to remove all stopped containers, dangling images, and unused networks. You can also use `docker image prune -a` to remove all unused images, not just dangling ones. Also make sure you run `docker volume prune` to remove all your volumes. Why go through this step? Ever since I started to use docker and minikube on my system, I've noticed that the free disk space on my `/` partition has been steadily decreasing. That's because all docker images, containers, volumes and layers are stored under /var/lib/docker/ In my case, /var is part of my `/` partition which is actually on a small NVMe drive. Due to the design of my motherboard, my NVMe slot is on the underside of my motherboard making maintenance and upgrading a hassle.

In any case, the location of docker images and volumes is configurable, and so for our development environment, lets change it. What we want to do is configure the Docker system daemon. Here we have some choices. We can either edit the service file at /lib/systemd/system/docker.service or edit the docker daemon configuration file at /etc/docker/daemon.json. A third option which could work for us is to use soft links.

{% hint style="info" %}
Read https://docs.docker.com/storage/storagedriver/ to understand more about storage driver options for docker. In my setup, I'm only using ext4 filesystems, but if you're really concerned with performance when building containers, you may want to look at alternatives such as overlay2.
{% endhint %}

The official docker daemon configuration page at [https://docs.docker.com/config/daemon](https://docs.docker.com/config/daemon) will tell you about the first two methods, while the third option is something that you'll find if you do a Google search with the phrase "how to change the default location for docker create volume". On ArchLinux, systemd is the most popular init system used; however, it is by no means the only one. So let's avoid changing a service file and instead look into editing `/etc/docker/daemon.json`. As the file extension suggests, this is a JSON file. Mine looks like this:

```bash
{
    "dns": ["192.168.10.1", "8.8.8.8"]
}
```

What we can do is change this file so that data asssociated with docker is on the a different partition, which has a lot more free disk space. In my case, I'll chose /home which is on an SSD, but it would have been ideal to choose a different location.

```bash
{
    "dns": ["192.168.10.1", "8.8.8.8"],
    "data-root": "/home/docker"
}
```

NOTE: Be very cautious about making these changes. You may want to make a backup of /etc/docker/daemon.json just in case.

What we are going to do is the following in terminal A:

```bash
$ su root
[root@localhost]# journalctl -f -u docker
```

In terminal B, we'll make the change:

```bash
$ su root
[root@localhost]# mv /var/lib/docker to /home/docker
[root@localhost]# systemctl restart dockerd
```

If you're unsure of the syntax of this file, I suggest that you look at the CommonConfig struct for docker at [https://raw.githubusercontent.com/docker/engine/master/daemon/config/config.go](https://raw.githubusercontent.com/docker/engine/master/daemon/config/config.go). You should see that the Root field is a string and is mapped to data-root.

Once done, try pulling and running hello-world to verify that docker is working.

## Managing Containers

You've no doubt used commands such as `docker images` and `docker containers` to manage docker resources on your machine. Let's look at some tools that can be used to accomplish the same and perhaps a little more.

If you use Visual Studio Code, there are a large number of extensions that let you manage docker. Most books and training courses don't talk about them, but you should explore them since they are very easy to install and use. The extension that I have is simply called the Docker extension and it also aids in validating the content of a Dockerfile.

[https://github.com/skanehira/docker.vim\[docker.vim](https://github.com/skanehira/docker.vim[docker.vim)\] is a plugin for managing docker using vim. You'll need to figure out how to install plugins for vim and then clone this repository to the plugins directory. Once installed, you can run commands such as `:DockerImages` to get a list of images.

[https://github.com/jesseduffield/lazydocker\[Lazydocker](https://github.com/jesseduffield/lazydocker[Lazydocker)\] is a standalone application that accomplishes much of the same but using a terminal window. If you want to know how to use the gocui library and how to create ArchLinux packages this is a good tool to investigate. It has has links at the bottom of the README.md to other docker terminal UIs.

I've tried all three, but I can't say with certainty that there is a clear winner. The docker.vim plugin is interesting, but for the amount of typing I need to do, doesn't feel like it's any better than using the command line. The Lazydocker application is a lot better, giving me the abiltiy to see everything related to docker in a kind of dashboard format. It took a while to navigate the sparse UI, but it does have lots of functionality. Finally, the docker extension for Visual Studio is the most pleasing to use, although it doesn't provide any metrics.

## Installing Minikube

Before installing minikube, we'll first make sure all our packages are up-to-date.

```bash
[root@localhost]# pacman -Syu
```

Reboot if you have major updates such as a new kernel.

Next, we install the minikube package.

```bash
[root@localhost]# pacman -S minikube
resolving dependencies...
looking for conflicting packages...

Packages (1) minikube-1.13.0-1

Total Download Size:   10.77 MiB
Total Installed Size:  47.45 MiB

:: Proceed with installation? [Y/n]
:: Retrieving packages...
 minikube-1.13.0-1-...    10.8 MiB  1698 KiB/s 00:06 [############################] 100%
(1/1) checking keys in keyring                       [############################] 100%
(1/1) checking package integrity                     [############################] 100%
(1/1) loading package files                          [############################] 100%
(1/1) checking for file conflicts                    [############################] 100%
(1/1) checking available disk space                  [############################] 100%
:: Processing package changes...
(1/1) installing minikube                            [############################] 100%
Optional dependencies for minikube
    kubectl: to manage the cluster [installed]
    virtualbox: to use --vm-driver=virtualbox [installed]
:: Running post-transaction hooks...
(1/1) Arming ConditionNeedsUpdate...
[root@localhost]#
```

You should also install kubectl if you haven't already; although minikube does provide the ability to run kubectl, it's very slow.

The first time you start minikube, it will create a directory at ~/.minikube, download a docker image and start a container. You can see it if you run docker images or use a tool such as the Docker extension for VSCode.

According to the documentation, it is possible to view the configuration for minikube by running `minikube config view`. However, when I first ran this command, nothing was returned: no default values, no warning messages, nothing. Then I had a hunch that I need to set the default driver to see any change, so I ran `minikube set driver docker`. This time the CLI returned a value!

```bash
$ minikube config view
$ minikube config set driver docker
❗  These changes will take effect upon a minikube delete and then a minikube start
$ minikube config view
- driver: docker
```

This confirms that, if you want to configure minikube, it's best to do it right after installation.

Let's poke around ~/.minikube to see where the driver was set. I found it under ~/.minikube/profiles/minikube/config.json. Searching for `docker` in this file, I found that the Driver field was set to `docker`. There are many other settings in this file. Of interest are the settings for memory, CPU and disksize, but we'll leave these alone. Perhaps we could revisit these later. Further down, I found a section for Nodes and Addons. These might be of interest later, so we'll make a note of these entries.

Now that we've configured minikube to use the docker driver by default, let's wrap up this section by making these changes permanent. When we start minikube a second time, we can confirm that minikube started with the docker driver as we requested.

```bash
$ minikube stop
...
$ minikube delete
$ minikube start
😄  minikube v1.13.0 on Arch
✨  Using the docker driver based on user configuration
👍  Starting control plane node minikube in cluster minikube
🔥  Creating docker container (CPUs=2, Memory=4000MB) ...
❗  This container is having trouble accessing https://k8s.gcr.io
💡  To pull new external images, you may need to configure a proxy: https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
🐳  Preparing Kubernetes v1.19.0 on Docker 19.03.8 ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube" by default
```
