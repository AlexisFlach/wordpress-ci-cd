### Continuous Integration / Continuos Deployment w/ Docker and Digital Ocean

###### docker-compose.yml

```yaml
version: '3.9'

services:
  db:
    image: mysql:8.0.26
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: wordpress
      MYSQL_PASSWORD: password
    networks:
      - wp
  phpmyadmin:
    depends_on:
      - db
    image: phpmyadmin/phpmyadmin
    restart: always
    ports:
      - '8080:80'
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: password 
    networks:
      - wp
  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - '${PORT}:80'
    restart: always
    volumes: ['./:/var/www/html']
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: password
    networks:
      - wp
networks:
  wp:
volumes:
  db_data:
```
###### github actions

**./github/workflows/digitaocean_depl.yml**

```yml
name: digitalocean_deploy

on:
  push:
    branches: [main]
    
jobs:
  Build:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@main
      - name: Create .env
        run: echo 'PORT=${{secrets.PORT }}' > .env 
      - name: Run build
        run: docker-compose up -d
```

Create a .env file

```
PORT=1234
```
###### github

Secrets -> New repository secret

```pseudocode
name PORT
value 80
```



###### Digital Ocean

**Create Droplets**

Ubuntu -> Marketplace -> Docker

```
ssh root@ip
```

Create a new user in order to run new files

```
adduser USER
```

Give user sudo priveleges

```
usermod -aG sudo USER
```

Login as another user

```
sudo su - USER
```



###### Add runner

Whenever the action/workflow runs a build, we want to know about in the VM.

repo -> settings -> actions -> runners -> add runner

1. Change os to Linux.
2. Copy all the steps from the **Download** section into VM and run step by step.

3. Copy all the steps from the **Configure** section into VM and run step by step.

   ```
   √ Settings Saved.
   ```

4. run runner

   ```
   ./run.sh
   ```

   ```
   Job Build completed with result: Failed
   PermissionError: [Errno 13] Permission denied
   ```

5. ```
   cd _work/folder/REPOFOLDER/REPOFOLDER
   cat .env
   PORT=80
   ```

###### Give newly created user Docker permission

```
exit
root@ip# sudo groupadd docker
groupadd: group 'docker' already exists
sudo usermod -aG docker USER
sudo su - USER

cd /actions_runners/
./run.sh

√ Connected to GitHub

Listening for Jobs

```

We will later change from using ./run.sh to the service file.

Go to github and rebuild job.

###### Make sure the runner is always running

```
cd ~/actions-runners
```

```
sudo ./svc.sh install
sudo ./svc.sh start

sudo ./svc.sh status
```



#### Thanks to:

##### The Tech Team:  https://www.youtube.com/watch?v=-nT1Xs7-qqA&ab_channel=TheTechTeam

##### Traversy Media:  https://www.youtube.com/watch?v=pYhLEV-sRpY&t=496s&ab_channel=TraversyMedia









