# Run portainer

```
docker run -d \
-p 9000:9000 \
--name=portainer \
--restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data \
portainer/portainer-ce:latest
```

## Ensuite dans "Stack" : 

```
version: "3"

services:
   guacd:
         container_name: guacd
         image: guacamole/guacd
         restart: unless-stopped
         networks:
            guac-net:
               ipv4_address: 172.30.0.2
            
   guacweb:
      container_name: guac_web
      image: guacamole/guacamole
      restart: unless-stopped
      ports:
         - 8080:8080
      environment:
         MYSQL_DATABASE: guacamole_db
         MYSQL_HOSTNAME: 172.30.0.3
         MYSQL_PASSWORD: !!!MOTDEPASSE!!!
         MYSQL_USER: guacamole_user
         GUACD_HOSTNAME: 172.30.0.2
      depends_on:
         - guacamole-sql
         - guacd
      networks:
         guac-net:
         
   guacamole-sql:
      container_name: guac-sql
      image: mysql
      restart: unless-stopped
      environment:
         MYSQL_ROOT_PASSWORD: !!!MOTDEPASSE!!!
      volumes:
         - dbdata:/var/lib/mysql
      networks:
         guac-net:
            ipv4_address: 172.30.0.3
            
volumes:
   dbdata:
    
networks:
   guac-net:
      driver: bridge
      ipam:
         driver: default
         config:
            - subnet: "172.30.0.0/16"
```

# Sur le docker mysql, dans la SGBD :

```mysql -u root -p
```
CREATE DATABASE guacamole_db;
```
```
CREATE USER 'guacamole_user'@'%' IDENTIFIED BY '!!!MOTDEPASSE!!!';
```
```
GRANT SELECT,INSERT,UPDATE,DELETE ON guacamole_db.* TO 'guacamole_user'@'%';
FLUSH PRIVILEGES;
```
## Then : 

```quit
```

# Sur le Serveur debian 

## Recuperer le fichier initdb.sql

```
docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --mysql > initdb.sql
```
```
docker cp ./initdb.sql guac-sql:/initdb.sql
```
# Sur le docker mysql, dans la SGBD :
```
use guacamole_db;
```
```
source ./initdb.sql
```
