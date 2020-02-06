# Docker Home Media Server - Docker Apps

This section will detail the installation and configuratio aof al the docker relatd aplications that are used tomake up our ultimate home media server.

## Portainer

<p align="center">
  <img src="https://pronto-core-cdn.prontomarketing.com/354/wp-content/uploads/sites/2/2018/11/volumes-1.png">
</p>

[Portainer](https://www.portainer.io/) provides a WebUI to manage all your docker containers. I strongly recommend this for newbies. Here is the code to add (copy-paste) in the docker-compose file (pay attention to blank spaces at the beginning of each line):

```yaml
portainer:
    image: portainer/portainer
    container_name: portainer
    restart: always
    command: -H unix:///var/run/docker.sock
    ports:
      - "XXXX:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ${USERID}/docker/portainer/data:/data
      - ${USERID}/docker/shared:/shared
    environment:
      - TZ=${TZ}
    networks:
      - traefik_proxy
    labels:
      traefik.enable: "true"
      traefik.home: "true"
      traefik.backend: portainer
      traefik.protocol: http
      traefik.port: 9000
      traefik.frontend.rule: Host:portainer.${DOMAINNAME}
      traefik.frontend.headers.SSLHost: portainer.${DOMAINNAME}
      traefik.docker.network: traefik_proxy
      traefik.frontend.passHostHeader: "true"
      traefik.frontend.headers.SSLForceHost: "true"
      traefik.frontend.headers.SSLRedirect: "true"
      traefik.frontend.headers.browserXSSFilter: "true"
      traefik.frontend.headers.contentTypeNosniff: "true"
      traefik.frontend.headers.forceSTSHeader: "true"
      traefik.frontend.headers.STSSeconds: 315360000
      traefik.frontend.headers.STSIncludeSubdomains: "true"
      traefik.frontend.headers.STSPreload: "true"
      traefik.frontend.headers.customResponseHeaders: X-Robots-Tag:noindex,nofollow,nosnippet,noarchive,notranslate,noimageindex
#      traefik.frontend.headers.frameDeny: "true" #customFrameOptionsValue overrides this
      traefik.frontend.headers.customFrameOptionsValue: 'allow-from https:${DOMAINNAME}'
      traefik.frontend.auth.forward.address: "http://oauth:4181"
      traefik.frontend.auth.forward.authResponseHeaders: X-Forwarded-User
      traefik.frontend.auth.forward.trustForwardHeader: "true"
```

### Replace/Configure:

```XXXX``` – port number on which you want the portainer Webui to be available at. It could be the same port as the container: 9000 (must be free). I stuck with ```9000```
After saving the ```docker-compose.yml``` file, run the following command to start the container and check if the app is accessible:

```bash
docker-compose -f ~/docker/docker-compose.yml up -d
```

Portainer WebUI should be available at http://SERVER-IP:XXXX. Repeat the above command after each container is added to ```docker-compose.yml``` file and ensure that the app works.

## Organizr

<p align="center">
  <img src="https://organizr.app/wp-content/uploads/2015/04/CJ8OziG-1024x552.jpg">
</p>

Home media server with several apps may be cool but now you will have to remember all the different port numbers to access them. That is where [Organizr](https://organizr.app/) comes in. Organizr provides a unified interface to access all your home server apps so you do not have to remember them individually. The tabbed interface allows you to work on your server with ease. You can even setup users and give them access to specific apps. The calendar provides an overview of what TV show episodes are coming soon.  Docker makes it easier to install. Here is the code to add in the docker-compose file (pay attention to blank spaces at the beginning of each line):

```yaml
organizr-v2:
    container_name: organizr-v2
    restart: always
    image: organizrtools/organizr-v2
    volumes:
      - ${USERDIR}/docker/organizrv2:/config
      - ${USERDIR}/docker/sharedv2:/shared
    ports:
      - "XXXX:80"
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
    networks:
      - traefik_proxy
    labels:
      - "traefik.enable=true"
      - "traefik.backend=organizr-v2"
      - "traefik.frontend.rule=Host:organizr.${DOMAINNAME}"
#      - "traefik.frontend.rule=Host:${DOMAINNAME}; PathPrefixStrip: /organizr"
      - "traefik.port=80"
      - "traefik.docker.network=traefik_proxy"
#      - "traefik.frontend.headers.SSLRedirect=true"
      - "traefik.frontend.headers.STSSeconds=315360000"
#      - "traefik.frontend.headers.browserXSSFilter=true"
#      - "traefik.frontend.headers.contentTypeNosniff=true"
#      - "traefik.frontend.headers.forceSTSHeader=true"
#      - "traefik.frontend.headers.SSLHost=${DOMAINNAME}"
#      - "traefik.frontend.headers.STSIncludeSubdomains=true"
#      - "traefik.frontend.headers.STSPreload=true"
#      - "traefik.frontend.headers.frameDeny=false"
      - "traefik.frontend.headers.customFrameOptionsValue=allow-from https://organizr.${DOMAINNAME}"
      - "traefik.frontend.passHostHeader=true"
```

### Replace/Configure:

```XXXX``` – port number on which you want the Organizr Webui to be available at. It could be the same port as the container: 80 (must be free). Port 80 is the default webserver port. So you do not need to specify the ```:80``` at the end of the IP Address or Domain Name. I used ```8841```
After saving the ```docker-compose.yml``` file, run the following command to start the container and check if the app is accessible:

```bash
docker-compose -f ~/docker/docker-compose.yml up -d
```

Organizr WebUI should be available at http://SERVER-IP:XXXX (:XXXX not needed if port 80 is used). 

## Maria DB - Database Server

<p align="center">
  <img src="https://mariadb.org/wp-content/uploads/2015/10/mariadb-usa-inc.png">
</p>

[MariaDB](https://mariadb.org/) is a community-developed fork of MySQL database system. It can serve as the central data location for all your apps that support it. In addition, for apps that save huge amount of data, a database such as MySQL can significantly improve performance over a file-based database such as SQLite (eg. Home Assistant, NextCloud).

```yaml
mariadb:
    image: "linuxserver/mariadb"
    container_name: "mariadb"
    hostname: mariadb
    volumes:
        - ${USERDIR}/docker/mariadb:/config
    ports:
      - target: 3306
        published: 3306
        protocol: tcp
        mode: host
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
    labels:
      - "traefik.enable=false"
    networks:
      - default
```

### Replace/Configure:

1. ```${USERDIR}/docker/mariadb``` – Path to you want to save your database files. ```${USERDIR}``` is filled automatically from the environment file we created previously.
2. ```Ports``` – I recommend leaving the default port 3306 as is unless you know what you are doing.
3. ```MYSQL_ROOT_PASSWORD``` – Filled in automatically from the environment file we created previously.

Save and run the ```docker-compose.yml``` file as described previously and check if the container is working.

## phpMyAdmin - Manage MariaDB

<p align="center">
  <img src="https://www.phpmyadmin.net/static/images/screenshots/users.png">
</p>

[phpMyAdmin](https://www.phpmyadmin.net/) is a free, open-source tool developed in PHP and intended to handle the administration of MySQL database management system (DBMS). It is designed to perform a wide range of operations on MySQL over the web. It offers the user-friendly web interface, support for most MySQL features, management of MySQL users and privileges, management of stored procedures and triggers, import and export of data from various sources, administration of multiple servers and much more.

```yaml
phpmyadmin:
    hostname: phpmyadmin
    container_name: phpmyadmin
    image: phpmyadmin/phpmyadmin
    restart: always
    links:
      - mariadb:db
    ports:
      - XXXX:80
    environment:
      - PMA_HOST=mariadb
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
    labels:
      traefik.enable: "true"
      traefik.home: "true"
      traefik.backend: phpmyadmin
      traefik.protocol: http
      traefik.port: 80
      traefik.frontend.rule: Host:pma.${DOMAINNAME}
      traefik.frontend.headers.SSLHost: pma.${DOMAINNAME}
      traefik.docker.network: traefik_proxy
      traefik.frontend.passHostHeader: "true"
      traefik.frontend.headers.SSLForceHost: "true"
      traefik.frontend.headers.SSLRedirect: "true"
      traefik.frontend.headers.browserXSSFilter: "true"
      traefik.frontend.headers.contentTypeNosniff: "true"
      traefik.frontend.headers.forceSTSHeader: "true"
      traefik.frontend.headers.STSSeconds: 315360000
      traefik.frontend.headers.STSIncludeSubdomains: "true"
      traefik.frontend.headers.STSPreload: "true"
      traefik.frontend.headers.customResponseHeaders: X-Robots-Tag:noindex,nofollow,nosnippet,noarchive,notranslate,noimageindex
#      traefik.frontend.headers.frameDeny: "true" #customFrameOptionsValue overrides this
      traefik.frontend.headers.customFrameOptionsValue: 'allow-from https:${DOMAINNAME}'
      traefik.frontend.auth.forward.address: "http://oauth:4181"
      traefik.frontend.auth.forward.authResponseHeaders: X-Forwarded-User
      traefik.frontend.auth.forward.trustForwardHeader: "true"
```

### Replace/Configure:

1. ```XXXX``` – I changed the default port to 8888.
2. ```MYSQL_ROOT_PASSWORD``` – Filled in automatically from the environment file we created previously.

Save and run the ```docker-compose.yml``` file as described previously and check if the container is working.
