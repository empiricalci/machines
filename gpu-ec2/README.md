
# GPU EC2

## AMI Details
AMI ID: ``ami-b1e2c4a6``

### Includes

This AMI includes
- Docker 1.12
- NVIDIA drivers 361.42
- [nvidia-docker](https://github.com/NVIDIA/nvidia-docker)

### Instance types

You can use the Launch Wizard to launch the AMI on the following instance types:

- p2.xlarge
- g2.2xlarge

### Test

On a p2.xlarge:
```
$ sudo nvidia-docker run --rm nvidia/cuda nvidia-smi
ubuntu@ip-172-31-55-142:~$ sudo nvidia-docker run --rm nvidia/cuda nvidia-smi
Fri Nov  4 22:51:48 2016       
+------------------------------------------------------+                       
| NVIDIA-SMI 361.42     Driver Version: 361.42         |                       
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K80           Off  | 0000:00:1E.0     Off |                    0 |
| N/A   51C    P8    28W / 149W |     22MiB / 11519MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
```

On a g2.2xlarge:
```
$ sudo nvidia-docker run --rm nvidia/cuda nvidia-smi
Fri Nov  4 22:24:49 2016       
+------------------------------------------------------+                       
| NVIDIA-SMI 361.42     Driver Version: 361.42         |                       
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GRID K520           Off  | 0000:00:03.0     Off |                  N/A |
| N/A   23C    P8    17W / 125W |     11MiB /  4095MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
```



## Building the AMI from Scratch

### Launching the Machine
Using the launch wizard on AWS  we select:
- **Base AMI**: ``ami-40d28157`` Ubuntu server 16.04 LTS 
- **Instance type:** ``g2.2xlarge``

### Installing the NVIDIA driver

[nvidia-cuda's Wiki](https://github.com/NVIDIA/nvidia-docker/wiki/Deploy-on-Amazon-EC2) provides good instructions
on how to install the drivers on EC2, however the instructions for blacklisting the noveau kernel module are not mentioned
so we take that section from 
[Caffe's Wiki](https://github.com/BVLC/caffe/wiki/Install-Caffe-on-EC2-from-scratch-(Ubuntu,-CUDA-7,-cuDNN-3)#installing-the-nvidia-drivers).

First, update the linux image to be compatible with NVIDIA's drivers:
```
sudo apt-get install linux-image-extra-virtual
```
#### Blacklist novueau
We now need to disable nouveau since it conflicts with NVIDIA's kernel module:
```
sudo vim /etc/modprobe.d/blacklist-nouveau.conf
```

And add the following lines to this file:
```
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias lbm-nouveau off
```
Back in the terminal/shell, execute the commands:
```Shell
echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf
sudo update-initramfs -u
sudo reboot
```

After the reboot is complete, we have a few more steps:
```Shell
sudo apt-get install linux-source
sudo apt-get install linux-headers-`uname -r`
```

#### NVIDIA drivers
```
# Install NVIDIA drivers 361.42
sudo apt-get install --no-install-recommends -y gcc make libc-dev
wget -P /tmp http://us.download.nvidia.com/XFree86/Linux-x86_64/361.42/NVIDIA-Linux-x86_64-361.42.run
sudo sh /tmp/NVIDIA-Linux-x86_64-361.42.run --silent
```

### Installing docker
From [Docker installation instructions](https://docs.docker.com/engine/installation/linux/ubuntulinux/):

Add ``APT`` sources and update.
```
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
echo deb https://apt.dockerproject.org/repo ubuntu-xenial main | sudo tee /etc/apt/sources.list.d/docker.list 
sudo apt-get update
```
Verify that there's a version of docker available for you to install. It should show  version 1.12
```
apt-cache policy docker-engine
```
Install docker-engine and start the daemon
```s
sudo apt-get install docker-engine
sudo service docker start
```
Verify it works
```
sudo docker run hello-world
```

### Installing nvidia-docker

```
# Install nvidia-docker and nvidia-docker-plugin
wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.0/nvidia-docker_1.0.0-1_amd64.deb
sudo dpkg -i /tmp/nvidia-docker*.deb && rm /tmp/nvidia-docker*.deb
```
Verify it works:
```
sudo nvidia-docker run --rm nvidia/cuda nvidia-smi
```


### Save the AMI

Use AWS UI to save the AMI.
