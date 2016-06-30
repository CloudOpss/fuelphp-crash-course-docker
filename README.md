# FuelPHP Crash Course Docker

## Table of Contents
  1. Contributors
  2. Introduction
    - Authors
    - Prerequisite Knowledge
    - Included Files
  3. Part 1: Getting Started (Installation and Setup)
    - Installing Virtual Box
    - Installing Docker Components with Homebrew
    - Setting Up the Machine
    - Testing Docker
  4. Part 2: Setting Up a LEMP Server
    - Docker Compose
    - Editing docker-file.yml
      - Version
      - Services
      - Image
      - Ports
      - Volumes
      - Links
      - Build and Dockerfiles
      - Environment
    - Creating a Dockerfile
      - From
      - Run
    - Configuring the LEMP Stack
    - Testing the LEMP Stack
  5. More practice

## Contributors
  - [Sam Belcastro](https://github.com/samuel-belcastro/)

## Introduction
Docker is an open platform that allows developers to use a virtualization
method called containerization. Containerization allows distributed software
to run without running an entire virtual machine for each application.

**Prerequisite Knowledge**

In order to be successful in comprehending this tutorial, having knowledge in
the following will help. The bolded items are especially important.
  - **HTML, CSS, Javascript, and PHP**
  - Github
  - **Nginx Configuration**
  - PHP Configuration
  - MySQL Configuration
  - FastCGI

**Included Files**

The files within this repository represent a completed version of this
tutorial.

The only files that must be downloaded are:

- *nginx.conf*
- *default.conf**

Everything else will be created during the tutorial.

## Part 1: Getting Started (Installation and Setup)
  Docker is built to work with Linux based Kernels. Therefore, a virtual machine is
  required to develop with Docker on Mac OSX. Luckily, Docker realizes the
  struggle involved with this, and makes using the virtual machine as easy as
  possible!

  1. Installing Virtual Box

    In this tutorial, we will be using Virtual Box. Navigate to
    http://www.virtualbox.org/ and download the latest version of Virtual Box.

  2. Installing Docker Components with Homebrew

    Docker offers a nice tool called Docker Toolbox, but, for this tutorial, we
    are going to get our hands dirty and dive directly into the terminal!

    The first Docker tool we are going to install is the docker engine.

    Install docker with the following command:
    ```
      $ brew install docker
    ```

    Next, we want to install docker machine. Docker machine will allow us to
    use docker seamlessly with Virtual Box.

    Install docker machine with the following command:
    ```
      $ brew install docker-machine
    ```

    Finally, we want to install docker compose. Docker compose allows us to
    build and link docker containers, along with mounting directories
    from the host machine to the containers, with a single .yaml file!

    After setting up docker compose, we will be able to launch our whole web
    server with one command!

    Install docker compose with the following command:
    ```
      $ brew install docker-compose
    ```

  5. Setting Up the Virtual Machine

    The first thing we want to do when setting up a machine is check which
    machines we have already created. To do this we simply type the following
    command:

    ```
      $ docker-machine ls

      NAME   ACTIVE   DRIVER  STATE   URL   SWARM   DOCKER   ERRORS
    ```

    There is nothing in the table output because we have not created any
    machines yet!

    In order to create our own machine we are going to run the following
    command:

    ```
      $ docker-machine create --driver virtualbox default
    ```

    Let's analyze this command:

    **docker-machine** - tells the OS to run the docker-machine program<br />
    **create** - calls the create subcommand, which creates a new virtual
    machine<br />
    **--driver virtualbox** - tells docker-machine that we are using Virtual
    Box<br />
    **default** - names the machine default<br />

    After creating the machine, run:

    ```
      $ docker-machine ls

      NAME      ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER   ERRORS
      default   *        virtualbox   Running   tcp://IP_OF_MACHINE:PORT            v1.11.2     
    ```
    Congratulations! The machine is now successfully created. There is now only
    one step left in order to seamlessly use our docker machine.

    Run the following command:

    ```
      $ eval "$(docker-machine env default)"
    ```

    This command will connect your shell to the new machine. Now every time
    you use the **docker** command, it will direct it towards the machine.
    We can replace *default* with any machine name.

    **This command must be ran every time you start the machine!**


  4. Testing Docker

    Now that we have successfully installed Docker and set up a machine, we
    will make sure the core engine is working.

    Run the command:
    ```
      $ docker run hello-world
    ```

    Let's analyze this command:<br />
      **docker** - Tells the OS that you are using the docker program<br />
      **run** - A subcommand that creates and runs a docker container<br />
      **hello-world** - The image to be loaded into the container<br />

    Each container requires an image to run. For example, our web server
    container is going to use Linux Alpine as the guest OS, and we can build
    our container with the Linux Alpine image.

    So, how does docker know where to get the image? Docker checks (in order):
      1. Local Images already installed through Docker
      2. **DockerHub** - public repository for Docker images

    Docker will then install the image if the image is on DockerHub.   

    For this tutorial, we pulled *hello-world* from DockerHub, and Docker built
    a container to hold and run it in. This program **is not** running on the
    host OS!

## Part 2: Setting Up a LEMP Server

  For this tutorial, LEMP stands for Linux Nginx MySQL PHP. We are going to
  create a stack of these services in order to serve our advanced web needs!

  Before Docker, a typical way to implement a LEMP stack would be loading a
  version of Linux to a physical or virtual machine, and independently
  installing each service with its respective environment variables and
  dependencies. If your super awesome web app became popular enough, you would
  probably need to scale, and to do that you would have to... haha good luck
  figuring that out!

  With Docker and its containerization, we can create independent services,
  running in their own little environment, with their own dependencies to
  worry about. They can also be scaled fairly easily.

  1. Docker Compose

  I know what you're thinking, "YES! WE FINALLY GET TO USE DOCKER COMPOSE!, "
  and right you are my friend... right you are.

  The tutorial above shows how to create a basic container with an image from
  Dockerhub, but it does not show how to install dependencies and edit config
  files within that container. While that can all be done in the command line,
  we don't have time for that!! We are going to knock out all of our container
  customization in a nice file named *docker-compose.yml*.

  To begin, let's open our terminal and run the command:

  ```
    $ mkdir myDocker
  ```

  This command can be ran in any directory, preferably a project directory.
  The directory can also have any desired name.

  Now that we have a directory for our project, let's create a
  *docker-compose.yml*.

  This file may be edited in any text editor, but it must reside the the
  root of the directory that was previously created.

  2. Editing docker-file.yml

  Now, we will work on the *docker-compose.yml* file one section at a time.
  **SPACES MATTER IN THIS FILE**

  **Version**

    The *docker-compose.yml* always start with a version number. For this
    tutorial, we will be using version 2.

    ```
      1| version: '2'
    ```

  **Services**

    Following version, we will begin to define each service we would like
    docker to create. This is where we design our containers.

    ```
      1| version: '2'
      2|
      3| services:
    ```

    For this tutorial, we will be creating a container for Nginx, PHP, and
    MySQL.

    We will start with Nginx

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
    ```

    **Image**

    For Nginx we will need an image. The image we are going to use is
    *nginx:stable-alpine* from Dockerhub

    This is the same as running the *docker run* command in the
    terminal. When the *docker-compose.yml* file is compiled and executed,
    the container being created will look for a local image and then proceed
    to Dockerhub if the local image does not exist.

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
      5|   image: nginx:stable-alpine
    ```

    **Ports**

    We will also need to set Nginx to listen to the host machine's port 80.
    Remember that the container is acting as it's own separate machine.

    So how do we link Nginx's port 80 to the host machines port 80? Linking
    the two ports is actually pretty simple!

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
      5|   image: nginx:stable-alpine
      6|   ports:
      7|    - "80:80"
    ```
    At this point, we have designed a container that will run an Nginx
    webserver.

    Open the terminal, and run the command:

    ```
      $ docker-compose up
    ```

    Docker Compose will look for the *docker-compose.yml* file, and then it
    will build the containers for the services we have listed in our file with
    the settings we have defined.

    If you run the *docker-machine ls* command, and type the IP of your
    machine (without the port) into your browser, you will see the default
    Nginx homepage!

    **Volumes**

    However, we are still missing one crucial component of web development. We
    need a place to store all of our files to be served by Nginx.

    Nginx creates a default directory for these files, but we still are not
    able to add or modify files in that directory efficiently. To solve this
    problem, we are going to map the directory created within our container to
    a directory within our host machine. This way we can edit our files in
    our favorite editor without needing to constantly upload our files to the
    container.

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
      5|   image: nginx:stable-alpine
      6|   ports:
      7|    - "80:80"
      8|   volumes:
      9|    - ./app/public:/var/www/html/public:ro
    ```

    Now, when we build this container, everything within our
    *myDocker/app/public* directory will go the */var/www/html/public*
    directory within the container. Each time we edit or add a file,
    the container will follow the same action. You just have to refresh the
    page!

    Being able to edit out config files for the web server would also be a
    nice addition to our container. To do this we will just map the specific
    config files in the same manner as we did with directory above.

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
      5|   image: nginx:stable-alpine
      6|   ports:
      7|    - "80:80"
      8|   volumes:
      9|    - ./app/public:/var/www/html/public:ro
     10|    - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
     11|    - ./config/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf:ro
    ```

    Here we linked two of Nginx's default configuration files. If we need to
    edit any of the default configuration files, we now have a way to do so
    from our host machine.

    As of right now, we have designed a container that acts as a fully
    functional web server that can serve HTML, CSS, and Javascript files. We
    also have the power to edit files right from our host machine. This
    includes project files, and configuration files.

    **Note: When editing configuration files, the web server must be restarted
    for changes to take effect. Restarting the web server is done by stopping
    the container in the terminal (control + C), and then re-composing the
    container (docker-compose up).**

    **Links**

    Finally, since we do not want to run the whole stack in a single container,
    we will need to tell the Nginx container to link to other containers.

    When Docker creates a new container, Docker assigns the container an IP
    address. When a container is stopped and ran again, the container gets
    assigned a different IP. It is a bad idea to link containers via their
    IP because of the inconsistency with the IP assignments. To combat this
    issue, Docker provides a hosts file where Docker links the container name
    to its IP. This allows us to link by name, and let Docker figure everything
    else out.  

    For this tutorial, we will be linking to a phpfpm container. This
    container will be explained further as we design the next service, but
    for now just add the link to your *docker-compose.yml* file.

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
      5|   image: nginx:stable-alpine
      6|   ports:
      7|    - "80:80"
      8|   volumes:
      9|    - ./app/public:/var/www/html/public:ro
     10|    - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
     11|    - ./config/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf:ro
     12|  links:
     12|    - phpfpm
    ```

    Now we will proceed to the phpfpm container.

    We require a phpfpm service for Nginx to turn to when Nginx receives a
    PHP request. Nginx does not know how to handle PHP, but Nginx is willing to
    learn how to direct PHP requests through some configuration, which will be
    addressed later. For now, we will design the PHP service.

    In summary, this service will receive a PHP request from Nginx through
    FastCGI, and return a response back to Nginx.

    **Build and Dockerfiles**

    We will begin by adding phpfpm to our list of services.

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
      5|   image: nginx:stable-alpine
      6|   ports:
      7|    - "80:80"
      8|   volumes:
      9|    - ./app/public:/var/www/html/public:ro
     10|    - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
     11|    - ./config/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf:ro
     12|   links:
     12|    - phpfpm
     13|  
     14|  phpfpm
    ```  

    PHP has many extensions, and we may not want to waste space and resources
    on ones we don't need. These extensions can also be very specific to what
    is trying to be accomplished. Docker allows us to create our own image
    based on already existent images. To do this we use a *Dockerfile*. We will
    create this *Dockerfile* after finishing our *docker-compose.yml*.

    To notify docker-compose that we are building our own image, we use *build*
    instead of *image*.  

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
      5|   image: nginx:stable-alpine
      6|   ports:
      7|    - "80:80"
      8|   volumes:
      9|    - ./app/public:/var/www/html/public:ro
     10|    - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
     11|    - ./config/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf:ro
     12|   links:
     13|    - phpfpm
     14|  
     15|  phpfpm
     16|   build:
    ```

    We also need to tell docker-compose where to find our Dockerfile, so
    docker-compose to correctly communicate to Docker where to build from.

    To do this, we give docker-compose a *context* and the path to the
    *Dockerfile* relative to that context.

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
      5|   image: nginx:stable-alpine
      6|   ports:
      7|    - "80:80"
      8|   volumes:
      9|    - ./app/public:/var/www/html/public:ro
     10|    - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
     11|    - ./config/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf:ro
     12|   links:
     13|    - phpfpm
     14|  
     15|  phpfpm
     16|   build:
     17|    context: .
     18|    dockerfile: docker_files/Dockerfile.phpfpm
     19|   ports:
     20|    - "9000:9000"
    ```

    FastCGI is going to communicate with phpfpm on port 9000, and, since we
    already defined and explained ports, we just tagged it on after build.

    **Note: For this tutorial, we only use one Dockerfile, but we are setting
    up the project as if we had 2 or more Dockerfiles.**

    Now we can move on to environment variables!!

    **Environment**

    We did not have to set environment variables for our Nginx service, but,
    since we are using MySQL, we will be required to set some for phpfpm.

    Defining environment variables is also a relatively simple process. We are
    going to define four variables, and they will define how we interact with
    our MySQL container.

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
      5|   image: nginx:stable-alpine
      6|   ports:
      7|    - "80:80"
      8|   volumes:
      9|    - ./app/public:/var/www/html/public:ro
     10|    - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
     11|    - ./config/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf:ro
     12|   links:
     13|    - phpfpm
     14|  
     15|  phpfpm
     16|   build:
     17|    context: .
     18|    dockerfile: docker_files/Dockerfile.phpfpm
     19|   ports:
     20|    - "9000:9000"
     21|   environment:
     22|    - DB_HOST=mysql
     23|    - DB_NAME=fuel
     24|    - DB_USER=user
     25|    - DB_PASS=secret
    ```

    Let's dissect these variables!

    **DB_HOST** - the address of the mysql server. Due to the inconsistency in
    IP assignment when creating docker containers, we use the name of the
    container instead of hardcoding an IP address.<br />
    **DB_NAME** - The name of the database that will be accessed<br />
    **DB_USER** - The user that will be accessing the database<br />
    **DB_PASS** - The password of the user accessing the database<br />

    **Note: This file now contains a password. Beware when uploading code to
    websites such as GitHub!!**

    To wrap up this container, we are going to add mounted volumes like we did
    with Nginx, and we will also add a link to the mysql container that we
    will be building next!

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
      5|   image: nginx:stable-alpine
      6|   ports:
      7|    - "80:80"
      8|   volumes:
      9|    - ./app/public:/var/www/html/public:ro
     10|    - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
     11|    - ./config/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf:ro
     12|   links:
     13|    - phpfpm
     14|  
     15|  phpfpm
     16|   build:
     17|    context: .
     18|    dockerfile: docker_files/Dockerfile.phpfpm
     19|   ports:
     20|    - "9000:9000"
     21|   environment:
     22|    - DB_HOST=mysql
     23|    - DB_NAME=fuel
     24|    - DB_USER=user
     25|    - DB_PASS=secret
     26|   volumes:
     27|    - ./app:/var/www/html:rw
     29|   links:
     30|    - mysql
    ```

    Finally, we are going to build a mysql container to handle all of our
    databasing.

    ```
      1| version: '2'
      2|
      3| services:
      4|  nginx:
      5|   image: nginx:stable-alpine
      6|   ports:
      7|    - "80:80"
      8|   volumes:
      9|    - ./app/public:/var/www/html/public:ro
     10|    - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
     11|    - ./config/nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf:ro
     12|   links:
     13|    - phpfpm
     14|  
     15|  phpfpm
     16|   build:
     17|    context: .
     18|    dockerfile: docker_files/Dockerfile.phpfpm
     19|   ports:
     20|    - "9000:9000"
     21|   environment:
     22|    - DB_HOST=mysql
     23|    - DB_NAME=fuel
     24|    - DB_USER=user
     25|    - DB_PASS=secret
     26|   volumes:
     27|    - ./app:/var/www/html:rw
     29|   links:
     30|    - mysql
     31|
     32|  mysql
     33|    image: mysql:5.5.47
     34|    environment:
     35|     - MYSQL_ROOT_PASSWORD=secret
     36|     - MYSQL_USER=user
     37|     - MYSQL_PASSWORD=secret
     38|     - MYSQL_DATABASE=fuel
     39|    ports:
     40|     - "3306:3306"
    ```

    Just to recap what is going on here:

    **image** : using mysql:5.5.47 image from Dockerhub to build our container<br />
    **environment** : setting environment variables<br />
    **ports** : telling the container to listen to the hosts port 3306 on its<br />
    3306 port  

    This concludes the creation of our *docker-compose.yml*! Keep in mind that
    this is only the design of out docker containers. They will not be created
    until we run the command:

    ```
      $ docker-compose up
    ```

    At this point, this command will generate an error because we have not
    created the Dockerfile that our phpfpm container is going to build from.

    That will be our next task!

  3. Creating a Dockerfile

    A *Dockerfile* is a file without any specific extension that Docker uses to
    automate the process of building custom images. Unless explicitly stated,
    Docker will look in the current directory for *Dockerfile* by default when
    running the command:

    ```
      $ docker build
    ```

    Since we thought ahead, and created a directory called *docker_builds*, we
    can decide to add more custom images later. We will place our docker file
    in *docker_builds*, and we will name our file *Dockerfile.phpfpm*. We are
    not required to add the *.phpfpm*, but it allows us to see which build
    is in the Dockerfile.  

    Finally, we can get to editing this Dockerfile!

    **From**

    Each Dockerfile requires a base image. These base images can come from
    Dockerhub, or a private repository. In our case, the image is *php:5-fpm*
    from Dockerhub, but an image can be anything as simple as a fresh linux
    based operating system.

    The base image is defined at the top of the Dockerfile:

    ```
      1| FROM php:5-fpm
    ```

    **Run**

    Next, we are going to install any custom programs and extensions we want
    to include for our specific container that are not in the base image.

    For this project we are going to need:

    **php5-mysql** - so we can communicate with our mysql container
    **git** - so we can pull projects from github if needed
    **zip / unzip** - allows us to use dependency managers such as Composer

    We will write in the Dockerfile to tell docker to run a command in our
    container upon creation that will install these programs.

    ```
      1| FROM php:5-fpm
      2|
      3| RUN apt-get -y update && apt-get install -y \
      4|   php5-mysql \
      5|   git \
      6|   zip \
      7|   unzip
    ```

    The -y tells *apt-get* to assume yes to all prompts when installing these
    programs.

    We could write this all in one line, but it is best practice to separate
    each program with this syntax. The programs being installed are made clear,
    and they are easily added and removed.

    Next we want to install the extensions, so we add another RUN keyword.

    ```
      1| FROM php:5-fpm
      2|
      3| RUN apt-get -y update && apt-get install -y \
      4|   php5-mysql \
      5|   git \
      6|   zip \
      7|   unzip
      9|
     10| RUN docker-php-ext-install pdo pdo_mysql
    ```

    These last two commands are optional.

    The first one installs Composer, a dependency manager for PHP MVC
    frameworks such as FuelPHP. The second one gives the web user permission to
    write to directories.

    ```
      1| FROM php:5-fpm
      2|
      3| RUN apt-get -y update && apt-get install -y \
      4|   php5-mysql \
      5|   git \
      6|   zip \
      7|   unzip
      9|
     10| RUN docker-php-ext-install pdo pdo_mysql
     11|
     12| RUN bash -c "curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer"
     13| RUN sed -i "s/^www-data:x:[0-9]*/www-data:x:1000/" /etc/passwd
    ```

    This concludes our Dockerfile!

  4. Configuring our LEMP stack

    This tutorial is intended to teach the basics of Docker. I have provided the
    configuration files in order for this LEMP Stack to work in the repository.

    There should two files (relative to the myDocker directory):

      *config/nginx/nginx.conf*<br />
      *config/nginx/conf.d/default.conf*

    These files should be in the exact same locations as they are in the
    repository, so our *docker-compose.yml* file can work.

  5. Testing the LEMP stack

    We have officially created a containerized LEMP stack, and we are ready to
    test it!!!!

    Go to the terminal, and type the following command:

    ```
      $ docker-machine ls
    ```

    Make sure your machine is running, and take note of the IP address
    (without the port).

    Assuming you are connected to your machine with

    ```
      $ eval "$(docker-machine env default)"
    ```

    you can run the command

    ```
      $ docker-compose up
    ```

    Docker will take a minute to build everything, but, after it finishes, go
    to your browser and type the IP of your machine.

    The Nginx default page should show up.

    Create a new file with the name: *index.php*

    Next write the following contents to the file:

    ```
      1| <?php
      2|    echo phpinfo();
      3| ?>
    ```

    Save the file to *app/public/* and refresh your page.

    You should see the entire php configuration file put into a nicely
    formatted webpage.

## More Practice

By this point, we have enough knowledge to learn how to create a FuelPHP web
application using docker to host all of the web services. Checkout this
[FuelPHP Crash Course w/ Docker](http://ucf.github.io/fuelphp-crash-course/)
tutorial to gain some more experience with docker while creating a practical
application! XD
