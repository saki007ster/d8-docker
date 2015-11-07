d8-docker
==================

## Setting up Drupal 8 environment using Docker toolbox on OS X

There are a lot of different ways to setup a local development environment, and there are usually many challenges along the way. One problem that often arises is that a developer’s local environment differs from their co-workers and/or their staging or production environments. For example, maybe you’re running PHP 5.6, your colleague is running PHP 5.5, and production is running PHP 5.4. This can cause issues when you share or deploy code that works in one environment but not in another. Using Docker, Docker Compose, and Bowline we can remove this pain point by ensuring that all of the environments are the same.
First off what are Docker, Docker toolbox

* Docker is “an open-source project that automates the deployment of applications inside software containers.” Essentially, Docker runs a processes from within a container that includes all dependencies the process needs to run. This allows containers to run almost anywhere. For example, you may have a container for running MySQL.
* Docker Toolbox - The Docker Toolbox is an installer to quickly and easily install and setup a Docker environment on your computer. Available for both Windows and Mac, the Toolbox installs Docker Client, Machine, Compose, Kitematic and VirtualBox.

By using these tools, you can ensure that each member of your team has the same local setup. That way, if code works in one environment, then it works in all of them.

Now I’ll go through the steps I followed to set everything up on my Mac. I’ll be showing you how to set up a fresh Drupal 8 install, however you can also use these tools on new or existing Drupal 6 and 7 projects.

This repo contains a recipe for making a Docker container running Drupal8, using Linux, Apache, MySQL, Memcache and SSH. You can also use it on the Drupal Sprints for quickly starting working on your Drupal8 git repo.

To use it, make sure you first [Install Docker](https://docs.docker.com/installation/).

### Download and install the docker toolbox from [here](https://www.docker.com/docker-toolbox)

Once you install Docker Toolbox you have two options in Applications folder
1. Kinematic (GUI)
2. Docker Terminal (CLI)

### From the Docker Terminal
1. Open the “Applications” folder or the “Launchpad”. 
2. Find the Docker Quickstart Terminal and double-click to launch it. The application:
    * opens a terminal window
    * creates a default VM if it doesn’t exists, and starts the VM after
    * points the terminal environment to this VM
3. Once the launch completes, the Docker Quickstart Terminal reports: 
Verify your setup succeeded by running the hello-world container.

```
$ docker run hello-world
```

### From your shell
This section assumes you are running a Bash shell. You may be running a different shell such as C Shell but the commands are the same.
#### Create a new Docker VM.

```
$ docker-machine create --driver virtualbox default
```

This creates a new default VM in VirtualBox.

The command also creates a machine configuration in the~/.docker/machine/machines/default directory. You only need to run the create command once. Then, you can use docker-machineto start, stop, query, and otherwise manage the VM from the command line.

List your available machines.

```
$ docker-machine ls
```

Get the environment commands for your new VM - default

```
$ docker-machine env default
```

this returns

```
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.101:2376"
export DOCKER_CERT_PATH="/Users/saket/.docker/machine/machines/default"
export DOCKER_MACHINE_NAME="default"
# Run this command to configure your shell:
# eval "$(docker-machine env default)"
```

then we run following common dos that we can map default virtualbox container to docker and run the docker commands in our terminal.

```
$ eval "$(docker-machine env default)”
```

## Get the image from this repo and run it using port 80:

```
docker run -i -t -p 80:80 saki007ster/drupal8
```
That's it!


# SPRINTING

If you want **Code and Database persistence** with an already existent Drupal8 code that you have on your computer, run it with:

```
cd; mkdir d8; cd d8

git clone --depth 5 --branch 8.0.x http://git.drupal.org/project/drupal.git

docker run -it \
--volume=$HOME/d8/mysql:/var/lib/mysql \
--volume=$HOME/d8/drupal:/var/www/html \
-p 80:80 -p 3306:3306 saki007ster/drupal8
```

You can remove the local settings.php and the mysql directory for a fresh Drupal8 install with existent code:

```
cd $HOME/d8
rm -rf mysql/ repo/sites/default/settings.php
```


### Credentials:
* Drupal account-name=admin & account-pass=admin
* ROOT SSH/MYSQL PASSWORD will be on /mysql-root-pw.txt
* DRUPAL   MYSQL_PASSWORD will be on /drupal-db-pw.txt

#### How to go back to the last docker run?
```
docker ps -al
(get the container ID)
sudo docker start -i -a (container ID)
```

### Example usage for testing:
Using docker exec {ID} {COMMAND}, to run your own commands.
```
~$ docker run --name mydrupal8 -i -t -p 80:80 saki007ster/drupal8

~$ docker exec mydrupal8  uptime
 10:02:59 up 16:41,  0 users,  load average: 1.17, 0.92, 0.76

~$ docker exec mydrupal8 drush status | head
 Drupal version         :  8.0.0-beta12
 Site URI               :  http://default
 Database driver        :  mysql
 Database hostname      :  localhost
 Database port          :  3306
 Drupal bootstrap       :  Successful
 ```

#### For older Drupal versions check:
https://github.com/saki007ster/d7-docker

#### You can also clone this repo somewhere and build it,
```
git clone https://github.com/saki007ster/d8-docker.git
cd d8-docker
sudo docker build -t <yourname>/drupal8 .
```
#### Or build it directly from github,
```
docker build -t saki007ster/drupal8 https://github.com/saki007ster/d8-docker.git
```

Note1: you cannot have port 80 already used or the container will not start.
In that case you can start by setting: `-p 8080:80`

Note2: To run the container in the background
```
sudo docker run -d -t -p 80:80 <yourname>/drupal8
```

## More docker awesomeness

This will create an ID that you can start/stop/commit changes:
```
# docker ps
ID             IMAGE                      COMMAND                   CREATED              STATUS              PORTS
dda662a2a3f9   <yourname>/drupal8:latest   /bin/bash /start.sh      3 minutes ago       Up 6 seconds        80->80
```

Start/Stop
```
sudo docker stop dda662a2a3f9
sudo docker start dda662a2a3f9
```

Commit the actual state to the image
```
sudo docker commit dda662a2a3f9 <yourname>/drupal8
```

Starting again with the commited changes
```
sudo docker run -d -t -p 80:80 <yourname>/drupal8 /start.sh
```

Shipping the container image elsewhere
```
sudo docker push  <yourname>/drupal8
```

You can find more images using the [Docker Index][docker_index].

### Clean up
While i am developing i use this to rm all old instances
```
sudo docker ps -a | awk '{print $1}' | grep -v CONTAINER | xargs -n1 -I {} sudo docker rm {}
```

### Known Issues
* Warning: This is still in development and ports shouldn't be open to the outside world.


## Contributing
Feel free to submit any bugs, requests, or fork and contribute to this code. :)

1. Fork the repo
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## Authors

Created and maintained by [Saket Kumar][author]
http://saki007ster.github.io

## License
GPL v3

[author]:                 https://github.com/saki007ster
[docker_index]:           https://index.docker.io/

