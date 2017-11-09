docker-cartodb
==============
This is a fork of sverhoeven/docker-cartodb with a few additions:
* Uses CartoDB "Classic" interface rather than "Carto Builder" for user Dev

---------------
[![](https://images.microbadger.com/badges/image/sverhoeven/cartodb.svg)](https://microbadger.com/#/images/sverhoeven/cartodb "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/version/sverhoeven/cartodb.svg)](https://hub.docker.com/r/sverhoeven/cartodb/)

This docker container provides a fully working cartodb development solution
without the installation hassle.

Just run the commands and then connect to http://cartodb.localhost with your browser.

The default login is dev/pass1234. You may want to change it when you run it for the outside.

It also creates an 'example' organization with owner login admin4example/pass1234.
Organization members can be created on http://cartodb.localhost/user/admin4example/organization

How to build the container:
---------------------------

```
git clone https://github.com/sverhoeven/docker-cartodb.git
docker build -t=sverhoeven/cartodb docker-cartodb/
```

The build uses the master branches of the [CartoDB GitHub repositories](https://github.com/CartoDB). A fresh build may fail when code requires newer dependencies then the Dockerfile provides or when code is not stable at the moment of building.

How to run the container:
-------------------------

```
docker run -d -p 80:80 -h cartodb.localhost sverhoeven/cartodb
```

The CartoDB instance has been configured with the hostname `cartodb.localhost`, this means the web browser and web server need to be able to resolve `cartodb.localhost` to an IP adress of the machine where the web server is running.
This can be done by adding cartodb.localhost alias to your hosts file. For example
```
sudo sh -c 'echo 127.0.1.1 cartodb.localhost >> /etc/hosts'
```
(For Windows it will be `C:\Windows\System32\drivers\etc\hosts`)

How to use a different hostname:
--------------------------------

For example to use `cartodb.example.com` as a hostname start with:
```
docker run -d -p 80:80 -h cartodb.example.com sverhoeven/cartodb
```

The chosen hostname should also resolve to an IP adress of the machine where the web server is running.

Persistent data
---------------

To persist the PostgreSQL data, the PostGreSQL data dir (/var/lib/postgresql) must be persisted outside the Cartodb Docker container.

The PostgreSQL data dir is filled during the building of this Docker image and must be copied to the local filesystem and then the container must be started with the local copy volume mounted.

```bash
docker create --name cartodb_pgdata sverhoeven/cartodb
# Change to directory to save the Postgresql data dir (cartodb_pgdata) of the CartoDB image
docker cp cartodb_pgdata:/var/lib/postgresql $PWD/cartodb_pgdata
docker rm -f cartodb_pgdata
# Inside container cartodb_pgdata is owned by postgres (uid=105) user,
# it should be owned by same user on the local filesystem
sudo chown -R 105.105 $PWD/cartodb_pgdata
```

After this the CartoDB container will have a database that stays filled after restarts.
The CartoDB container can be started with
```
docker run -d -p 80:80 -h cartodb.example.com -v $PWD/cartodb_pgdata:/var/lib/postgresql sverhoeven/cartodb
```

Geocoder
--------

The external geocoders like heremaps or mapzen have dummy api keys and do not work.
No attempts have been made or will be made in this Docker image to get the external geocoders to work.

The internal geocoder is configured, but contains no data inside the image.

To fill the internal geocoder run
```
docker exec -ti <carto docker container id> bash -c /cartodb/script/fill_geocoder.sh
```

This will run the scripts described at https://github.com/CartoDB/data-services/tree/master/geocoder
It will use at least 5.7+7.8Gb of diskspace to download the dumps and import them.
