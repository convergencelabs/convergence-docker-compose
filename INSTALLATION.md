# Setting up a CentOS 7 machine for use with Convergence

We recommend a `t3.large` ([specs](https://aws.amazon.com/ec2/instance-types/t3/)) or equivalent instance at minimum.  If on AWS, use the `CentOS Linux 7 x86_64 HVM EBS ENA 1803` AMI.

## Ports

TCP ports `443` (and optionally `80`) should accept inbound TCP traffic.

## Install Docker and docker-compose

You may need to use `sudo` with many of these commands.

```
# Add the Docker repo 
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker and its dependencies
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2 \
  docker-ce \
  epel-release

# Allow the new `docker` user to run commands without sudo
usermod -aG docker centos

# Start up docker
systemctl start docker

# Install docker-compose with PIP, a Python package manager
yum upgrade -y python*
yum install -y python-pip
pip install docker-compose

# Bump up memory allocation for elasticsearch
sudo sysctl -w vm.max_map_count=262144
```

At this point, Docker and docker-compose should be running.  Execute `docker` and `docker-compose` to make sure they were installed properly.

## Stand up Convergence

First, copy the contents of this repository up to the server.  Then follow the instructions in the README to stand up Convergence.
