# MIGRATION TO THE СLOUD WITH CONTAINERIZATION. PART 1 – DOCKER &AMP; DOCKER COMPOSE

This project introduces us to containerization, here we will learn about the major containerization app which is DOCKER.

## What is Docker

Docker is a platform that allows developers to package, distribute, and run applications in containers. Containers are lightweight, portable, and self-contained environments that can run on any system with the Docker engine installed, regardless of the underlying infrastructure. Docker provides a way to package an application along with all of its dependencies, libraries, and configuration files into a single, easy-to-manage unit. This makes it easier to deploy and scale applications across different environments, such as development, testing, and production. Docker also enables the creation of microservices-based architectures, where complex applications are broken down into smaller, independent components that can be developed, deployed, and managed separately.

Docker installation on different OS: **https://docs.docker.com/engine/install/**

## Building MYSQL container

**Method 1.**
1. Pull mysql docker image from docker registry:

```markdown
docker pull mysql/mysql-server:latest

docker image ls

```
2. Build mysql container from the downloaded image: Since it is a db, root password will be needed to login to db:

```markdown
docker run --name boma-mysql -e MYSQL_ROOT_PASSWORD=Password1 -d mysql/mysql-server:latest

docker ps -a
OR
docker container ls
```
3. Connecting directly to the container running the MySQL server:

```markdown
$ docker exec -it boma-mysql bash

or

$ docker exec -it boma-mysql mysql -uroot -p

```

Connection to mysql successful.

Image of connection:

"D:\connection-to-mysql-container.png"

Flags explained:

* exec: used to execute a command from bash itself
* -it: makes the execution interactive and allocate a pseudo-TTY
* bash: this is a unix shell and its used as an entry-point to interact with our container
* mysql: The second mysql in the command "docker exec -it mysql mysql -uroot -p" serves as the entry point to interact with mysql container just like bash or sh
* -u: mysql username
* -p: mysql password
* -d: Instruct container to run run in the background


**Method 2**

## 1. Create docker network

```markdown
docker network create --subnet=10.0.0.0/24 tooling_app_network

docker network ls
```

tooling_app_network: netwok name

Creating environment variable to store our root password

$ export MYSQL_PW=Password1 with powershell: $Env:MYSQL_PW = "Password1" OR $MYSQL_PW = "Password1"
echo $MYSQL_PW OR $env:MYSQL_PW

## 2. Pull the image and run container with our newly created network:

```markdown
$ docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest

docker ps -a 
```

**Flags used**

-d: runs the container in detached mode
--network: connects a container to a network
-h: specifies a hostname

## 3. Create Mysql user for remote login:

Create a file and name it create_user.sql and add the below code in the file:

 ```markdown
 $ CREATE USER 'Admin'@'%' IDENTIFIED BY 'Password1'; GRANT ALL PRIVILEGES ON * . * TO 'toolingdb'@'%'; 

 ```
Run the script:
Ensure you are in the directory create_user.sql file is located or declare a path

 ```markdown
 $ docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < create_user.sql
 ```
 The above code login to the mysql container and run the script to create new db user 


## 4. Connecting to the MySQL server from a second container running the MySQL client utility

Run the MySQL Client Container:

```markdown
docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u  -p
```
**Image of mysql client:**

![alt text]("D:\MY DESKTOP\CLOUD WORLD\Dare DevOps Project\DARE.live videos\Project20\Clientmysql.png")

**Flags Explained:**
--name gives the container a name
-it runs in interactive mode and Allocate a pseudo-TTY
--rm automatically removes the container when it exits
--network: connects a container to a network
-h a MySQL flag specifying the MySQL server Container hostname
-u user created from the SQL script
admin username-for-user-created-from-the-SQL-script-create_user.sql
-p password specified for the user created from the SQL script

Prepare database schema
Now you need to prepare a database schema so that the Tooling application can connect to it.

Clone the Tooling-app repository from here
 $ git clone https://github.com/darey-devops/tooling.git 

On your terminal, export the location of the SQL file
 $ export tooling_db_schema=/tooling_db_schema.sql 

You can find the tooling_db_schema.sql in the tooling/html/tooling_db_schema.sql folder of cloned repo.

Verify that the path is exported

 echo $tooling_db_schema
Use the SQL script to create the database and prepare the schema. With the docker exec command, you can execute a command in a running container.
 $ docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema 

Update the .env file with connection details to the database

## 5. Prepare database schema
Now you need to prepare a database schema so that the Tooling application can connect to it.

1. Clone the Tooling-app repository from here
 $ git clone https://github.com/darey-devops/tooling.git 

2. On your terminal, export the location of the SQL file
 $ export tooling_db_schema=tooling/html/tooling_db_schema.sql 

You can find the tooling_db_schema.sql in the tooling/html/tooling_db_schema.sql folder of cloned repo.

Verify that the path is exported

 echo $tooling_db_schema

3. Use the SQL script to create the database and prepare the schema. With the docker exec command, you can execute a command in a running container.
 $ docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema 

4. Update the .env file with connection details to the database

The .env file is located in the html tooling/html/.env folder but not visible in terminal. you can use vi or nano

```markdown
sudo vi .env

MYSQL_IP=mysqlserverhost
MYSQL_USER=username
MYSQL_PASS=client-secrete-password
MYSQL_DBNAME=toolingdb
```
**Flags used:**

MYSQL_IP: mysql ip address "leave as mysqlserverhost"
MYSQL_USER: mysql username for user export as environment variable
MYSQL_PASS: mysql password for the user exported as environment varaible
MYSQL_DBNAME: mysql databse name "toolingdb"

**Image of db tooling schema:**

5. Run the Tooling App

Containerization of an application starts with creation of a file with a special name - 'Dockerfile' (without any extensions). This can be considered as a 'recipe' or 'instruction' that tells Docker how to pack your application into a container. In this project, you will build your container from a pre-created Dockerfile, but as a DevOps, you must also be able to write Dockerfiles.

You can watch this video to get an idea how to create your Dockerfile and build a container from it.

And on this page, you can find official Docker best practices for writing Dockerfiles.

So, let us containerize our Tooling application; here is the plan:

Make sure you have checked out your Tooling repo to your machine with Docker engine
First, we need to build the Docker image the tooling app will use. The Tooling repo you cloned above has a Dockerfile for this purpose. Explore it and make sure you understand the code inside it.
Run docker build command
Launch the container with docker run
Try to access your application via port exposed from a container
Let us begin:

Ensure you are inside the directory "tooling" that has the file Dockerfile and build your container :

```markdown
 $ docker build -t tooling:0.0.1 . 
```

In the above command, we specify a parameter **-t**, so that the image can be tagged tooling"0.0.1 - Also, you have to notice the **.** at the end. This is important as that tells Docker to locate the Dockerfile in the current directory you are running the command. Otherwise, you would need to specify the absolute path to the Dockerfile.

**6. Run the container:**
 
 ```markdown
 $ docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1 
 ```

Let us observe those flags in the command.

We need to specify the **--network** flag so that both the Tooling app and the database can easily connect on the same virtual network we created earlier.
The **-p** flag is used to map the container port with the host port. Within the container, apache is the webserver running and, by default, it listens on port 80. You can confirm this with the CMD ["start-apache"] section of the Dockerfile. But we cannot directly use port 80 on our host machine because it is already in use. The workaround is to use another port that is not used by the host machine. In our case, port 8085 is free, so we can map that to port 80 running in the container.

## Note: You will get an error. But you must troubleshoot this error and fix it. Below is your error message.

AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 172.18.0.3. Set the 'ServerName' directive globally to suppress this message
Hint: You must have faced this error in some of the past projects. It is time to begin to put your skills to good use. Simply do a google search of the error message, and figure out where to update the configuration file to get the error out of your way.

If everything works, you can open the browser and type http://localhost:8085

Final page result: 


The default email is test@gmail.com, the password is 12345 or you can check users' credentials stored in the toolingdb.user table.