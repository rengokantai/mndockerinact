#### mndockerinact
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
