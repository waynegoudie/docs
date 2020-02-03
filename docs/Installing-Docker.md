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

