## Docker Port Assignment

Now that all the basic setup is done we can start with the easy part: setting up apps through containers. I have broken this section down into few different categories. What apps you choose for each category is a matter of personal preference. I have shown the setup instruction for the apps I recommend. Where possible I have also provided some popular alternatives.

It is very important that you pay attention to the blank spaces in the code snippets below. You will have to define a port number for many of the web apps below. If a port is already used you will get an error when you run ```docker-compose.yml``` file. You can also manually see all the ports that are currently listening/taken using the following command:

```bash
sudo netstat -tulpn | grep LISTEN
```

Before we start adding containers, I recommend making a list of 10-12 free ports that you can remember easily. For example, 9100 to 9112, if you find that they are free.

Note that ```${$USERDIR}```, ```${PUID}```, ```${PGID}```, ```${TZ}```, ```${MYSQL_ROOT_PASSWORD}```, etc. in the Traefik docker compose example code blocks will be automatically filled by the compose file from the environment file we created / edited previously. If you did not set environmental variables, you can replace them with actual values in the code blocks below.

In the docker compose snippets below you will notice that ```${USERDIR}/docker/shared``` is setup as a volume in almost all of the containers. The idea is to use this a shared folder between all of the containers for things like SSL certificates etc. if you have them. If you do not have them, nothing wrong in leaving the line as is. In the advanced guide, we will see how to add your SSL certificates for secure and trusted connection to your apps.
