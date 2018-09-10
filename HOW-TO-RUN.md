# How to Run

This is a step-by-step guide how to run the example:

## Installation

* The example is implemented in Java. See
   https://www.java.com/en/download/help/download_options.xml . The
   examples need to be compiled so you need to install a JDK (Java
   Development Kit). A JRE (Java Runtime Environment) is not
   sufficient. After the installation you should be able to execute
   `java` and `javac` on the command line.

* The example run in Docker Containers. You need to install Docker
  Community Edition, see https://www.docker.com/community-edition/
  . You should be able to run `docker` after the installation.

* The example need a lot of RAM. You should configure Docker to use 4
  GB of RAM. Otherwise Docker containers might be killed due to lack
  of RAM. On Windows and macOS you can find the RAM setting in the
  Docker application under Preferences/ Advanced.
  
* After installing Docker you should also be able to run
  `docker-compose`. If this is not possible, you might need to install
  it separately. See https://docs.docker.com/compose/install/ .

## Build

Change to the directory `microservice-consul-demo` and run `./mvnw clean
package` or `mvnw.cmd clean package` (Windows). This will take a while:

```
[~/microservice-consuldns/microservice-consuldns-demo]./mvnw clean package
....
[INFO] 
[INFO] --- maven-jar-plugin:2.6:jar (default-jar) @ microservice-consuldns-demo-order ---
[INFO] Building jar: /Users/wolff/microservice-consuldns/microservice-consuldns-demo/microservice-consuldns-demo-order/target/microservice-consuldns-demo-order-0.0.1-SNAPSHOT.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:1.4.5.RELEASE:repackage (default) @ microservice-consuldns-demo-order ---
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] microservice-consuldns-demo ........................... SUCCESS [  1.401 s]
[INFO] microservice-consuldns-demo-hystrix-dashboard ......... SUCCESS [  3.601 s]
[INFO] microservice-consuldns-demo-customer .................. SUCCESS [ 25.636 s]
[INFO] microservice-consuldns-demo-catalog ................... SUCCESS [ 36.618 s]
[INFO] microservice-consuldns-demo-order ..................... SUCCESS [ 27.781 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:35 min
[INFO] Finished at: 2017-09-07T18:08:13+02:00
[INFO] Final Memory: 52M/416M
[INFO] ------------------------------------------------------------------------
```

If this does not work:

* Ensure that `settings.xml` in the directory `.m2` in your home
directory contains no configuration for a specific Maven repo. If in
doubt: delete the file.

* The tests use some ports on the local machine. Make sure that no
server runs in the background.

* Skip the tests: `./mvnw clean package 
  -Dmaven.test.skip=true` or `mvnw.cmd clean package
  -Dmaven.test.skip=true` (Windows).

* In rare cases dependencies might not be downloaded correctly. In
  that case: Remove the directory `repository` in the directory `.m2`
  in your home directory. Note that this means all dependencies will
  be downloaded again.

## Run the containers

First you need to build the Docker images. Change to the directory
`docker` and run `docker-compose build`. This will download some base
images, install software into Docker images and will therefore take
its time:

```
[~/microservice-consuldns/docker]docker-compose build 
....
Removing intermediate container 1d59f8227b12
Step 4/4 : EXPOSE 8989
 ---> Running in 11e7fbacfa01
 ---> 9cfa7772986f
Removing intermediate container 11e7fbacfa01
Successfully built 9cfa7772986f
Successfully tagged msconsuldns_hystrix-dashboard:latest
```

Afterwards the Docker images should have been created. They have the prefix
`msconsuldns`:

```
[~/microservice-consuldns/docker]docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
msconsuldns_hystrix-dashboard   latest              9cfa7772986f        49 seconds ago      197MB
msconsuldns_order               latest              12b279e78975        52 seconds ago      225MB
msconsuldns_apache              latest              22fac099ba93        55 seconds ago      255MB
msconsuldns_catalog             latest              c23c535ecaf6        2 minutes ago       225MB
msconsuldns_customer            latest              a780e4f49bac        2 minutes ago       225MB
```

Figure out the IP adress of your system and set it to the environment
variable CONSUL_HOST. This is needed to ensure that the containers are
able to use Consul as a DNS server.

```
[~/microservice-consuldns/docker]ifconfig 
....
en5: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
....
	inet 192.168.1.152 netmask 0xffffff00 broadcast 192.168.1.255
....
[~/microservice-consuldns/docker]export CONSUL_HOST=192.168.1.152
```


Now you can start the containers using `docker-compose up -d`. The
`-d` option means that the containers will be started in the
background and won't output their stdout to the command line:

```
[~/microservice-consuldns/docker]docker-compose up -d
Creating network "msconsuldns_default" with the default driver
Pulling consul (consul:0.7.2)...
0.7.2: Pulling from library/consul
b7f33cc0b48e: Already exists
a4ca795d20eb: Pull complete
76bc5ef06918: Pull complete
965e633cb8c2: Pull complete
64e424fcbe65: Pull complete
Digest: sha256:ce15f85417a0cf121d943563dedb873c7d6c26e9b1e8b47bc2f1b5a3e27498e1
Status: Downloaded newer image for consul:0.7.2
Creating msconsuldns_consul_1 ... 
Creating msconsuldns_consul_1 ... done
Creating msconsuldns_order_1 ... 
Creating msconsuldns_catalog_1 ... 
Creating msconsuldns_customer_1 ... 
Creating msconsuldns_hystrix-dashboard_1 ... 
Creating msconsuldns_apache_1 ... 
Creating msconsuldns_hystrix-dashboard_1
Creating msconsuldns_order_1
Creating msconsuldns_catalog_1
Creating msconsuldns_apache_1
Creating msconsuldns_apache_1 ... done
```

As you can see the Consul Docker image is downloaded now.

Check wether all containers are running:

```
[~/microservice-consuldns/docker]docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                                                                              NAMES
2697aa91700b        msconsuldns_apache              "/bin/sh -c '/usr/..."   4 minutes ago       Up 4 minutes        0.0.0.0:8080->80/tcp                                                                               msconsuldns_apache_1
948f2576b0b0        msconsuldns_customer            "/bin/sh -c '/usr/..."   4 minutes ago       Up 4 minutes        8080/tcp                                                                                           msconsuldns_customer_1
0574e8dc5b11        msconsuldns_order               "/bin/sh -c '/usr/..."   4 minutes ago       Up 4 minutes        8080/tcp                                                                                           msconsuldns_order_1
144542583a05        msconsuldns_catalog             "/bin/sh -c '/usr/..."   4 minutes ago       Up 4 minutes        8080/tcp                                                                                           msconsuldns_catalog_1
15968668c5e8        msconsuldns_hystrix-dashboard   "/bin/sh -c '/usr/..."   4 minutes ago       Up 4 minutes        0.0.0.0:8989->8989/tcp                                                                             msconsuldns_hystrix-dashboard_1
c28d2a38f657        consul:0.7.2                 "docker-entrypoint..."   4 minutes ago       Up 4 minutes        8300-8302/tcp, 8400/tcp, 8600/tcp, 8301-8302/udp, 0.0.0.0:8500->8500/tcp, 0.0.0.0:8600->8600/udp   msconsuldns_consul_1
```
`docker ps -a`  also shows the terminated Docker containers. That is
useful to see Docker containers that crashed rigth after they started.

If one of the containers is not running, you can look at its logs using
e.g.  `docker logs msconsuldns_apache_1`. The name of the container is
given in the last column of the output of `docker ps`. Looking at the
logs even works after the container has been
terminated. If the log says that the container has been `killed`, you
need to increase the RAM assigned to Docker to e.g. 4GB. On Windows
and macOS you can find the RAM setting in the Docker application under
Preferences/ Advanced.
  
If you need to do more trouble shooting open a shell in the container
using e.g. `docker exec -it msconsuldns_catalog_1 /bin/sh` or execute
command using `docker exec msconsuldns_catalog_1 /bin/ls`.

You can access the microservices at http://localhost:8080/ , the
Hystrix dashboard on http://localhost:8989/ and the Consul dashboard
at http://localhost:8500 .

You can terminate all containers using `docker-compose down`.

If the order container cannot find the other containers, make sure
that no firewall is running on the Docker host. At least on Mac OS X
the firewall blocks the DNS requests.
