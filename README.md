#### mndockerinact
if seems like
```
docker run -it --name vv ubuntu /bin/sh = docker run -it --name vv ubuntu
docker run -it --name vv ubuntu /bin/bash
```
some commands easy to mess up
```
docker run --name ke ...
docker tag 12324  rengokantai/name:tag  / Or: docker tag rengokantai/name:tag1  rengokantai/name:tag2
docker build --tag/-t rengokantai/name .
```
CMD, ENTRYPOINT,RUN
```
ENTRYPOINT: same as docker container entrypoint
CMD: ENTRYPOINT other arguments
RUN: pre install softwares..
```
#####2
get all ps from a container
```
docker run -d --name namea busybox:latest /bin/sh -c "sleep 1000"
docker exec namea ps
```

get all pids
```
docker run --pid host busybox:latest ps
```

start a second nginx process,err
```
docker run -d --name webconflict nginx:latest
docker logs webconflict  //nothing
docker exec webconflict nginx -g 'daemon off;'
```
rename a container
```
docker run -d --name a1 nginx
docker rename a1 b1
```
create a container without running
```
docker create nginx
```

get latest created container,with truncated id
```
docker ps --latest(-l) --quiet(-q)  //full = --no-trunc
```

create readonly containr
```
docker run --name wp --read-only wordpress:4
```
inspect
```
docker inspect --format "{{.State.Running}}" wp
```
this is incorrect. set a db container
```
docker run -d --name wpdb -e MYSQL_ROOT_PASSWORD=root mysql:5   //= --env, inject env variable
```
then create a new wp,link to db
```
docker run -d --name wp2 --link wpdb:mysql -p 80 --read-only wordpress:4
```
but all file system will be locked.So we should unlock some area.
```
docker run -d --name wp3 --link wpdb:mysql -p 80 -v /run/lock/apache2 -v /run/apache2 --read-only wordpress:4
```


docker restart
```
docker run -d --name back --restart always busybox date
docker logs -f back
docker exec back echo test   //fail, because the container is not running
```
force kill container:
```
docker rm -f
dokcer kill
```

remove all container
```
docker rm -vf $(docker ps -a -q)
```
#####3 finding and installing software
save a image to tar file.
```
docker pull busybox:latest
docker save -o file.tar busybox:latest
```
load this tar to image
```
docker load -i file.tar
```

#####
create volume with cassandra
```
docker run -d --volume /var/lib/cassandra/data --name cass-shared alpine echo data
docker run -d --volumes-from cass-shared --name cass1 cassandra:2.2
docker run -it --rm --link cass1:cass cassandra:2.2 cqlsh cass1    //Or --link cass1
```
test
```
create keyspace ke with replication={ 'class':'SimpleStrategy','replication_factor':1};
select * from system.schema_keyspaces where keyspace_name = 'ke';
```
remove cass1, add cass2 to test whether the data still exist
```
docker rm -vf cass1
docker run -d --volumes-from cass-shared --name cass2 cassandra:2.2
docker run -it --rm --link cass1:cass cassandra:2.2 cqlsh cass2
```
then test the data still exists
```
select * from system.schema_keyspaces where keyspace_name = 'ke';
```
######4.2 bind mount volumes
```
docker run -d --name kw -v ~/kw:/usr/local/apache2/htdocs -p 80:80 httpd
```
set a readonly volume
```
docker run --rm -v ~/kw:/test:ro alpine /bin/sh -c 'echo test>/test/test'   //will fail because the volume is read only
```
test mount volumes
```
docker run --rm -v ~/kw:/test:ro alpine /bin/sh -c 'mount | grep test'
```
######Docker-managed volume
specify volume mount point inside container
```
docker run -d --volume /var/lib/cassandra/data --name cass-shared alpine echo data
```

override using --enrtypoint
```
docker run --name cat -d -v ~/logs:/data dockerinaction/ch4_writer_a
docker run --name wolf -d -v ~/logs:/data dockerinaction/ch4_writer_b
docker run --rm --entrypoint head -v ~/logs:/data alpine /data/logA
docker run --rm --entrypoint head -v ~/logs:/data alpine /data/logB  //same as below
docker run --rm  -v ~/logs:/data alpine head /data/logB
```
#####chap5 network
######
--net
```
docker run --rm --net none alpine:latest ip addr
```
ping test
```
docker run --rm --net none alpine:latest ping -w 2 8.8.8.8   //max time=2s
```

bridge
```
docker run --rm --net bridge alpine:latest ip addr
```
ping test
```
docker run --rm alpine:latest ping -w 2 8.8.8.8   //do not need to specify
```
--hostname --dns --dns-search
```
docker run --rm --hostname ke alpine:latest nslookup ke
```
```
docker run --rm --dns 8.8.8.8 alpine:latest nslookup github.com
```
```
docker run --rm --dns-search github.com alpine:latest nslookup developer
```

--add-host
```
docker run --rm --hostname ke --add-host y.com 127.0.0.1 alpine cat /etc/hosts
```
-P: expose all ports 
```
docker run -d --name ke -P
```
--expose add anotherport
```
docker run -d --name ke --expose 8000 -P box
```
check all opening ports
```
docker port ke
```

isolate inter container communitaion:
```
docker -d --icc=false
```
open container
```
docker run --rm --net host alpine:latest ip addr
```

another expose test (not working)
```
docker run -d --name data --expose 3306 dockerinaction/mysql_noauth service mysql_noauth start
docker run -d --name app --link data:db dockerinaction/ch5_web startapp.sh -db tcp://db:3306
docker run -d --name program 
```

env modification
```
docker run -d --name db --expose 2222 --expose 3333 --expose 4444/udp alpine:latest nc -l 0.0.0.0/2222
```
create link and list env var
```
docker run -it --rm --link db:database dockerinaction/ch5_ff env
```
#####6
run as user,first, get all available user: assign $1 to $0
```
docker run --rm  busybox awk -F: '$0=$2' /etc/passwd
```
```
docker run --rm -u nobody:nogroup busybox id
```
#####7
######7.1 build docker images from a container
basic:
```
docker run --name ke ubuntu touch /test
```
syntax: commit oldimage newimage
```
docker commit ke newimage
```
test:
```
docker rm -vf ke
docker run --rm newimage ls -l /test
```
######7.1.3
check a container's diff after making some change
```
docker run -it --name ke ubuntu:latest /bin/bash
touch a
exit
docker diff ke
```

Add a image name
```
docker commit -a "@authorname" -m "message" ke newname
```

new image can call `git version` directly. using --entrypoint
```
docker run --name ke --entrypoint git boxname
docker commit -a "@ke" -m "" ke boxname
docker rm -vf ke
docker run --name ke boxname version   // call git version
```
entrypoint: old way
```
 docker run --name ke -e ENV=ke busybox
 docker commit ke new
 docker run --rm new /bin/sh -c "echo \$ENV"
 ```
 entrypoint:
```
 docker run --name ke --entrypoint "/bin/sh" new -c "echo \$ENV"
 docker commit ke new
 docker run --rm new
 ```
 
 docker tag
 ```
 docker tag 1234 rengokantai/k:latest
 docker tag rengokantai/k:latest rengokantai/j  //create a new image, same id
 ```
 docker tag -f is deprecated.  
 
 using docker history
 ```
 docker history -H busybox
 ```
 ######7.3 exporting and importing flat
 export file system
 ```
 docker run --name ke busybox echo xx
 docker export --output ke.tar ke
 tar -tf ke.tar
 ```
#####cp8
######Dockerfile
using -f as input
```
docker build -t tagname -f filename .
```
######8.3 Injecting downstream build-time
```
ONBUILD RUN ls -la /app
```
#####ch9
######9.3.1 Using the registry image
```
docker run -d -p 5000:5000 -v "$(pwd)"/data:/tmp --restart-always --name ke registry:2
```

#####11
######docker-cmpose,log
```
docker-compose logs
docker-compose logs pump elasticserach
```
start part of service
```
docker-compose up -d db
```
by default, it docker compose service will start wieh dependency
```
docker-compose up --no-dep -d db
```
get all process of a service
```
docker-compose ps coffee
```
scale a service,and open multiple ps
```
docker-compose scale coffee=5
```

(to be)

#####chap12 Clusters with Machine and swarm
######12.1
```
docker-machine create --driver/-d virtubox h1
docker-machine ls
docker-machine inspect --format "{{.Driver.IPAddress}}" host1
docker-machine upgrade host1
docker-machine ssh  host1
```

copy between machines
```
docker-machine scp host1:/file host2:/file
docker-machine stop/kill/start host1
```
eval and powershell alternative
```
eval `docker-machine env host1`
docker-machine env --shell=powershell host1 | Invoke-Expression
```
list active machine
```
docker-machine active
docker-machine ls
```

