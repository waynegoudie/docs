## Docker Installation

For simplicity, we will install Docker and Docker Compose from the repository. Ubuntu has Docker in the official repository. However, this is can be several versions old. So we are going to install them from Docker repositories. First, prepare to add Docker repository using the following command:

```bash
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
```

Then, add the Docker repository's GPG key for verification of repository:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - 
```

Next, add the Docker repository:

```bash 
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" 
```
> There are different stable builds depending on the version of Ubunutu. Check [here](https://download.docker.com/linux/ubuntu/dists/bionic/pool/stable/amd64/) for the correct version.

Then, refresh Ubuntu packages list:

```bash
sudo apt-get update
```

If you did not encounter any errors (you won't if you follow the steps above correctly) during the above steps you should be good to install Docker and Docker Compose on Ubuntu using the following command:

```bash
sudo apt-get install docker-ce 
```

You can check the installed version using the command docker –version. Finally, test your docker setup using the following command.

```bash
sudo docker run hello-world
```

It will download a test container and run it. You should see an out similar to the one below:

<p align="center">
  <img src="https://www.smarthomebeginner.com/images/2018/04/docker-hello-world.png">
</p>

## Docker Compose Installation

In this guide I am going to use Docker compose to simplify installation of home server apps and reduce commandline work. Docker compose is in the Ubuntu repositories but it is quite old, as is the case most of the time. So let's install the latest version of Docker compose on Ubuntu.

First, find out the [latest version of Docker compose](https://github.com/docker/compose/releases) that is now available. Latest version at the time of writing this is ```1.25.4```

Next, install the latest version of Docker compose using the following command:

```bash
sudo curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```
Replace ```1.25.4``` with the currently available version as determined above. Finally, provide execute permissions to Docker Compose using the following command:

```bash
sudo chmod +x /usr/local/bin/docker-compose
```
If correctly installed you should see the version number as output for this command: ```docker-compose --version.```

## Add Linux User to Docker Group

Running and managing docker containers requires ```sudo``` privileges. So this means you will have to type sudo for every command or switch to the root user account. But you can get around this by adding the current user to the ```docker``` group using the following command:

```bash
sudo usermod -aG docker ${USER}
```

While this can be a minor security risk, the chances are very less and this is not an enterprise level setup but a home setup. So I recommend doing this for convenience.

## Setup Environmental Variables for Docker
Next, we are going to set some environmental variables such as timezone, user id, user group, etc. that docker containers should use. Create / edit the environmental variables file using the following command:

```bash
sudo nano /etc/environment
```

Add the following as separate lines at the end of the file:

```bash
PUID=1000
PGID=1000
TZ="America/New_York"
USERDIR="/home/USER"
MYSQL_ROOT_PASSWORD="password"
```

There will be more environment variables added, but these will be explained under each docker app configuration.

#### Replace/Configure:

1. ```PUID``` and ```PGID``` – the user ID of the linux user, who we want to run the home server apps as, and group ID of docker. Both of these can be obtained using the ```id``` command as shown below.

<p align="center">
  <img src="https://www.smarthomebeginner.com/images/2018/04/environmental-variables-for-docker.png">
</p>

In this guide, we are going to use 1000 for PUID, which is the user id of user and 140, which is the group id of docker group.

2. ```TZ``` – the timezone that you want to set for your containers. Get your TZ from this [timezone database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).
3. ```USERDIR``` – the path to the path to the home folder of the current user. You can also get this using the following command:
```bash
cd ~ ; pwd
```
4. ```MYSQL_ROOT_PASSWORD``` – MySQL administrator password for MariaDB and phpMyAdmin.

These environmental variables will be referred using {} throughout the docker compose file. You do not need to replace them. Their values will be automatically pulled from the environment file that we created / edited above.

> You will need to logout and log back in for the environmental variables to take effect.

That's it, the basic prep work to build our docker home server is done.

## BASIC DOCKER AND DOCKER COMPOSE PRIMER
Now let us start with a basic intro to Docker and Docker Compose. This is very important so you know what we are doing, when/how to stop and test, and when/how to start again. Then we are going to setup our ```docker-compose.yml``` file. Once our compose file is completely built, we will run it and you will see how in minutes your docker based home media server will be built. You can use any text editor to create your compose file. Make sure to follow ```yml``` syntax thoroughly as even differences in character spacing can throw errors. If you copy-paste from this guide, you should be fine. I am going to use ```nano``` editor.

### Docker Folder and Permissions
For simplicity, I created a folder called ```docker``` in my home folder. All my docker stuff, apps, app data will be stored in this container:

```bash
mkdir ~/docker
```

Next, let us setup appropriate permissions to the docker folder to avoid any permission error issues. Use the following commands in sequence:

```bash
sudo setfacl -Rdm g:docker:rwx ~/docker
sudo chmod -R 775 ~/docker
```

The above command forces any new sub-folders within the docker folder to inherit permissions from the docker folder. Some may disagree with the liberal permissions above but again this is for home use and it is restrictive enough.

### Starting Docker Compose File

Finally, let us start creating our ```docker-compose.yml``` file:

```bash
nano ~/docker/docker-compose.yml
```

Add the following two lines to it:

```yaml
version: "3.6"
services:
```

At any time, you can save and exit by pressing ```Ctrl + X``` -> ```y``` -> ```Enter``` and reopen for editing with the above nano command.

### Starting Containers using Docker Compose

This section is an intro to some of the commands you will use later in this guide. Running them at this point in the guide will throw errors. After adding compose options for each container (note that we have not added these yet), I recommend saving, exiting, and running the compose file using the following command to check if the container app starts correctly.

```bash
docker-compose -f ~/docker/docker-compose.yml up -d
```

The ```-d``` option daemonizes it in the background. Without it, you will see real-time logs, which is another way of making sure no errors are thrown. Press ```Ctrl + C``` to exit out of the real-time logs.

NOTE: At this point, you do not have any services.

### See Docker Containers

At any time, you can check all the docker containers you have on your system (both running and stopped) using the following command:

```bash
docker ps -a
```

As an example here is a list of my containers for now. “STATUS” column shows whether a container is running (for how long) or exited. The last column shows the friendly name of the container.

<p align="center">
  <img src="https://www.smarthomebeginner.com/images/2018/04/docker-list-of-containers-740x63.png?ezimgfmt=ng:webp/ngcb3">
</p>
Docker List of Containers


### Check Docker Container Logs

If you want to check the real-time logs while the container starts you can use the following command:

```bash
docker-compose logs 
```

In addition, you can also specify the name of the specific container at the end of the previous command if you want to see logs of a specific container. Here is a screenshot of the docker logs for my ```transmission-vpn``` container that was generated using the following command:

```bash
docker-compose logs transmission-vpn
```
<p align="center">
  <img src="https://www.smarthomebeginner.com/images/2018/04/docker-compose-logs-screenshot-740x66.png?ezimgfmt=ng:webp/ngcb3">
</p>
Docker Compose Real-time Logs for Containers


At any time, you can exit from the real-time logs screen by pressing ```Ctrl + C```.

### Stopping / Restarting Containers using Docker Compose

To stop any running docker container, use the following command:

```bash
docker-compose stop CONTAINER-NAME
```

Replace ```CONTAINER-NAME``` with the friendly name of the container. You can also replace ```stop``` with ```restart```. To completely stop and remove containers, images, volumes, and networks (go back to how it was before running docker compose file), use the following command:

```bash
docker-compose -f ~/docker/docker-compose.yml down
```

### Docker Cleanup
Remember, one of the biggest benefits of Docker is that it is extremely hard to mess up your host operating system. So you can create and destroy containers at will. But over time leftover Docker images, containers, and volumes can take several GBs of space. So at any time you can run the following clean up scripts and re-run your docker-compose as described above.

```bash
docker system prune
docker image prune
docker volume prune
```

These commands will remove any stray containers, volumes, and images that are not running or are not associated with any containers. Remember, even if you remove something that was needed you can always recreate them by just running the docker compose file.
