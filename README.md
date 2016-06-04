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
docker run -d --name wpdb -e MYSQL_ROOT_PASSWORD=ch2demo mysql:5   //= --env, inject env variable
```
then create a new wp,link to db
```
docker run -d --name wp2 --link wpdb:mysql -p 80 --read-only wordpress:4
```
