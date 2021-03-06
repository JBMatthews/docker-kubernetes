## Installing Docker Eco 
**Objective:** This lesson will serve as a guide for the installation of Docker<br>
**Preparation:** Host machine terminal and access to the internet<br>
**Outcome:** Each student will have Docker installed and tested<br>
**Data Files:** Ask Instructor.<br>

#### Step 1. Verify Installed Docker Eco Services

1. Run each of the following commands to determine if Docker, or other Docker components are already installed on your host machine.

```console
$ docker -v
$ docker-machine -v
$ docker-compose -v
```

#### Step 2. Determine Method of Installation

Please work with your instructor to determine which method we will best for you. Below we have listed the three most common installation solutions used currently, since Docker 1.13. For more methods of installation, visit ```https://docs.docker.com/engine/installation/```, which is Docker’s official page for installing Docker.

##### Method 1. Linux (Ubuntu) Host Installation

This method is appropriated for the following Ubuntu OS Versions:<br>
- Yakkety 16.10<br>
- *Xenial 16.04 (LTS)<br>
- Trusty 14.04 (LTS)<br>

> NOTE: This course was written using Xenial 16.04, and that is what is recommended.

1. Install the ```linux-image-extra-*``` packages, to allow Docker use of the aufs storage drivers. (Check with your instructor below installing these packages.)

```console
$ sudo apt-get update
$ sudo apt-get install -y --no-install-recommends linux-image-extra-$(uname -r) linux-image-extra-virtual
```

Docker can be installed so many different ways, depending on your needs. First, we’ll set up the Docker repository. This allows us to later install, update, or downgrade Docker from this same repository.
 
2. Set up the repository and install packages to allow apt to use a repository over HTTPS:

```console
$ sudo apt-get install -y --no-install-recommends apt-transport-https ca-certificates curl software-properties-common
```

3. Add Docker’s official GPG key:

```console
$ curl -fsSL https://apt.dockerproject.org/gpg | sudo apt-key add -
```

4. Verify that the key ID is ```58118E89F3A912897C070ADBF76221572C52609D```.

```console
$ apt-key fingerprint 58118E89F3A912897C070ADBF76221572C52609D
Output:
pub   4096R/2C52609D 2015-07-14
        Key fingerprint = 5811 8E89 F3A9 1289 7C07  0ADB F762 2157 2C52 609D
  uid    Docker Release Tool (releasedocker) <docker@docker.com>
```

5. Now, set up the stable repository using the following commands:

>NOTE: The ```lsb_release -cs``` sub-command below must return the name of your distribution:

```console
$ sudo add-apt-repository "deb https://apt.dockerproject.org/repo/ ubuntu-$(lsb_release -cs) main"
```

6. Update the ```apt``` package index.

```console
$ sudo apt-get update
```

7. Install the latest version of Docker, or go to the next step to install a specific version. Any existing installation of Docker is replaced. Use this command to install the latest version of Docker:

```console
$ sudo apt-get -y install docker-ce
```

##### Method 2. Windows Installation

Windows installation is one of the easiest ways to install Docker, since Docker’s 1.12 update.<br>

1. To do so, you will need to visit the official Docker Windows Installation page at: ```https://docs.docker.com/docker-for-windows/install/```. Follow along with the guide and be sure to install the current stable version of Docker being offered.<br> 

Once you have completed all the steps listed there, Docker is officially installed. Congrats!

##### Method 3. Mac OS X Installation 
Along with Windows’ Docker installation process, installing Docker on your Mac is one of the easiest options. No, you won’t need Terminal. Yet!<br>

1. Visit the official Docker Mac Installation page at: ```https://docs.docker.com/docker-for-mac/install/```. There you find a complete guide for installing Docker, through the GUI. Follow all the steps listed there and be sure to download the current stable version of Docker.<br>
Once you are done with all those steps, Docker is officially installed on your Mac. Congrats!

#### Step 3. Get Rid of the Sudo Problem
Docker needs to run with the correct permissions, which means typing ```sudo``` a lot. We can save ourselves a lot of time and energy by correcting the cause. Following along below:

1. Create a docker group:

```console
$ sudo groupadd docker
```

2. Add your user to the docker group:

```console
$ sudo gpasswd -a USERNAME docker
```

Log out and back in, then restart the Docker daemon.

###### Ubuntu

```console
$ sudo service docker resart
```

###### Mac OSX
Ask your instructor how to move ahead, if you are using a Mac.

###### Windows 

Like Mac OSX, consult with your instructor before moving ahead with this step.

#### Step 4. Finally Verification of Docker Installation

Since we are done with the process of installation, before going on, we would be wise to test it's functionality now. 

1. To verify that we’ve been successful run the following command: 

```console
$ docker run busybox:latest echo hello-world
```

Great job! You just ran your first Docker container.

#### Step 5. Installing Docker Machine

###### DOCKER FOR WINDOWS

Window's users will be relieved to hear that since Docker 1.12, ```docker-machine``` comes already installed and fully functional. Verify this by running:

```console
$ docker-machine --help
```

###### DOCKER FOR MAC
Mac OS X users, like Window's users can verify that ```docker-machine``` is already installed and fully functional by running:

```console
$ docker-machine --help
```

###### LINUX 

Linux is the only operation system that Docker requires a additional install process for Docker Machine, after installing Docker Engine. The following steps will ensure that  Docker Machine is properly installed onto your Linux distribution

1. Make sure you have successfully complete Step 2, Method 1 for installing Docker on Linux. If you have not, go back to complete that first, or ask your instructor for assistence

2. After you have completed installing Docker and correcting the sudo issue, run the following command to download the Docker Machine binary and extract it to your PATH:

```console
$ curl -L https://github.com/docker/machine/releases/download/v0.10.0/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine &&
  chmod +x /tmp/docker-machine &&
  sudo cp /tmp/docker-machine /usr/local/bin/docker-machine
```

3. Check the installation success, by running the following command:

```console
$ docker-machine version
```

Output:

```console
docker-machine version 0.10.0, build 76ed2a6
```

#### Step 6. Installing Docker Compose

1. Install Docker, if you haven't already in Step 2.

2. Download the Docker Machine binary and extract it to your PATH.

###### DOCKER FOR WINDOWS

- Run the following command to verify whether this service has been installed already, before moving to the next step:

```console
$ docker-compose --help
```

###### DOCKER FOR MAC

Mac OS X users, like Window's users can verify that ```docker-compose``` is already installed and fully functional by running:

```console
$ docker-compose --help
```

###### LINUX 

Linux is the only OS that requires an installation of Swarm, after installing Docker. Please, run the following command, which will do everything needed to get Compose install, up and fully functional. 

1. Install Docker Engine, if you haven't already early:

```console
$ docker --version
```
2. Now, run the following command to pull the binary code we need:

```console
$ curl -L https://github.com/docker/compose/releases/download/1.13.0-rc1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

3. Apply executable permissions to the binary:

```console
$ chmod +x /usr/local/bin/docker-compose
```

>NOTE: If you get a “Permission denied” error, your ```/usr/local/bin``` directory probably isn’t writable and you’ll need to install Compose as the superuser. Run ```sudo -i```, then the two commands below, then ```exit```.

4. Let's now attempt to confirm that we have successfully installed Docker Compose correctly:

```console
$ docker-compose --version
```

Output:

```console
docker-compose version 1.12.0, build b31ff33
```

#### Conclusion
Feel free to ask your instructor to help you confirm you have been successful, but if you are reading this you most likely have. What you have done is no small feat. Be proud of yourself. You have now, installed Docker Engine, Docker Machine, Docker Compose, and set sudo privledges over Docker. Now, let's move on, because we are prepared for whatever comes next. Great job!
