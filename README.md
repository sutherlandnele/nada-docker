# nada-docker
The docker image contains:

- Apache 2.4
- PHP 8.2
- MySQL 5.7

Follow these steps to configure NADA and run the docker container.

1. Setup folders to host NADA source and the Docker files

```
nada_docker 
|
| (clone the current repository)
|
| php.ini
| .docker/Dockerfile
| vhost.conf
| database.php
| ...
|
└───nada (nada source code)
│   │  ...
│   │
```

To download the docker and docker source code and setup the project folder structure using CLI, run these commands:.

```bash
#clone repo 
git clone https://github.com/mah0001/nada-docker.git nada_docker

#switch to project folder
cd nada_docker

#downoad nada source code into a subfolder
git clone https://github.com/ihsn/nada.git

#copy database.php file to `application/config/database.php`
cp database.php nada/application/config/database.php
```

Before you can start the docker container, review the `docker-composer.yml` to make sure all settings are correct:

Apache port: The default is set to 8383, you may want to change that to port 80.

```yaml
    ports:
      - "127.0.0.1:8383:80"
```

Mount volume: If you are using the project folder structure as above, there is no change needed. Otherwise, update the volume to map your local nada folder.

```yaml
    volumes:
      - ./nada:/var/www/html/nada
```

## Start docker container

From the main folder where you have extracted the docker files, run this from command line:

```bash
docker compose up 

```

That should build the image and launch the containers. If everything works, you should be able to see the NADA installer at http://localhost:8383/.

### Persist the MySQL data

#### Create a new directy called mysql-data under nada_docker directory

```bash
mkdir mysql-data
```

#### Stop the MySQL docker container

```bash
docker stop mysql
```

#### Copy the existing MySQL database files from within the container as initial data

```bash
sudo docker cp mysql:/var/local/mysql ./mysql-data
```

#### Update the docker-compose.yml file

Updated file should look like the following below with new volume mount point for the MySQL database files.

```bash
version: '3'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: nada_docker
    ports:
      - "127.0.0.1:8383:80"
    volumes:
      - ./nada:/var/www/html/nada
    links:
      - mysql
    environment:
      DB_HOST: mysql
      DB_DATABASE: nada_docker
      DB_USERNAME: nada_user
      DB_PASSWORD: nada_password      

  mysql:
    image: mysql:5.7
    container_name: mysql-57
    restart: always
    ports:
      - 13306:3306
    environment:
      MYSQL_DATABASE: nada_docker
      MYSQL_USER: nada_user
      MYSQL_PASSWORD: nada_password
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - ./mysql-data:/var/lib/mysql
```

#### Remove the docker container and start it back up

Open a new terminal and enter the commands below.

```bash
docker rm mysql
```

```bash
docker compose up
```

Now when the mysql database container is stopped and removed, the data is persisted on the mounted volume directory.