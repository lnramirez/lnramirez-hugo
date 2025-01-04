+++
date = "2016-07-18T17:28:12-04:00"
title = "And now we are using Hugo"

+++

A quick story on this site is as follow.

1. Created a simple website using spring, mongo, dojo and hosted in Cloudfoundry
2. Moved it to Openshift
3. Reimplemented frontend using clojurescript
4. Never updated it much
5. Some feautures stopped working
6. Decided to focus development somewhere else
7. Decided to use github pages and Hugo to host the blog
8. Here we are


Hugo makes sense to me because all my articles were written in markdown before,
though there are a couple of things that don't work the same, it shouldn't be that hard.

Now what I needed shouldn't be that difficult either

- All the articles where store in a collection `blogEntry` so I only had to get the data out of mongo
- And map it to a Hugo equivalent. Hugo is pretty simple:
    - you create a file .md and name of the file becomes url
    - file content has a header and what not but mainly you write markdown
- Make Hugo generate site content and upload it to github pages
- Finally update my cname record to point to github instead of redhat and tada this (will) should be up and running

So I had to endure a couple of things worth writting about.

- How I got the data out of Openshift
- How I generated the content

I found it so difficult to connect from a remote machine to Openshift, I am sure I saw somewhere people doing tunneling
or something in that direction. But in the end the data was the only thing needed. So I use RockMongo and generated a .js files.

I load it to a local mongo database since it would be easier (or so I thought). Then I have come to an age where
having to install stuff sounds too much, and since Docker is so hip nowadays and trying to learn new stuff gave it a chance.

Running docker with boot2docker in a Mac always is a pain. So there's a lot of indirection, for example I had to use Docker
in Virtual Box, once you follow all the guidelines for Mac you always forget how to connect from your local machine to Docker.
So have handy this cli always.

    $ eval $(docker-machine env default)

given that default is the name of virtual box machine
Now trying the line was torture

    $ docker run --name database -v /Users/luisrm/prj/per/docker/var/mongodata:/data/db -d mongo

It's supposed to run but then when was verifying that it was up and running

    $ docker ps
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

Bamn, nada! Driving me crazy. So more cli you can look at the logs

    $ docker logs -f database
    2016-07-18T20:35:46.508+0000 I CONTROL  [initandlisten] MongoDB starting : pid=1 port=27017 dbpath=/data/db 64-bit host=a8faaa94bdc6
    2016-07-18T20:35:46.508+0000 I CONTROL  [initandlisten] db version v3.2.8
    2016-07-18T20:35:46.508+0000 I CONTROL  [initandlisten] git version: ed70e33130c977bda0024c125b56d159573dbaf0
    2016-07-18T20:35:46.508+0000 I CONTROL  [initandlisten] OpenSSL version: OpenSSL 1.0.1e 11 Feb 2013
    2016-07-18T20:35:46.508+0000 I CONTROL  [initandlisten] allocator: tcmalloc
    2016-07-18T20:35:46.508+0000 I CONTROL  [initandlisten] modules: none
    2016-07-18T20:35:46.508+0000 I CONTROL  [initandlisten] build environment:
    2016-07-18T20:35:46.508+0000 I CONTROL  [initandlisten]     distmod: debian71
    2016-07-18T20:35:46.508+0000 I CONTROL  [initandlisten]     distarch: x86_64
    2016-07-18T20:35:46.508+0000 I CONTROL  [initandlisten]     target_arch: x86_64
    2016-07-18T20:35:46.508+0000 I CONTROL  [initandlisten] options: {}
    2016-07-18T20:35:46.529+0000 I STORAGE  [initandlisten] exception in initAndListen: 98 Unable to create/open lock file: /data/db/mongod.lock errno:13 Permission denied Is a mongod instance already running?, terminating
    2016-07-18T20:35:46.529+0000 I CONTROL  [initandlisten] dbexit:  rc: 100

now that gave me some shed on what's going on, apparently though it was mounted permissions and what non sense still couldn't acquired the lock.
Doing some stack overflow research I ended up on github bugs mentioning that mounting directly wouldn't work but you could create a docker volume
(whatever that means)

    $ docker volume create --name mongodbdata

and finally you run it with the volume you created and works

    $ docker run --name database -v /Users/luisrm/prj/per/docker/var/mongodata:/data/db -d mongo
    ae380df78141
    $ docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
    ae380df78141        mongo               "/entrypoint.sh mongo"   4 seconds ago       Up 3 seconds        0.0.0.0:27017->27017/tcp   database

Now you have to remember one thing, this container is not running on your machine, is listening on port 27017 but of the Virtual Box machine

    $ mongo localhost:27017
    MongoDB shell version: 3.2.0
    connecting to: localhost:27017/test
    2016-07-20T10:31:23.328-0400 W NETWORK  [thread1] Failed to connect to 127.0.0.1:27017, reason: errno:61 Connection refused
    2016-07-20T10:31:23.328-0400 E QUERY    [thread1] Error: couldn't connect to server localhost:27017, connection attempt failed :
    connect@src/mongo/shell/mongo.js:226:14
    @(connect):1:6

    exception: connect failed

So in order for you to connect to it you have to forward that port to something else.

* On virtual Box, right click the machine (default in my case) and click settings
* Click on network tab
* Click on Port forwarding
* Click on the + icon
* On the row select tcp protocol
* Enter 27017 for Guest Port
* And I chose 1111 for Host Port

then when you try

    $ mongo localhost:1111
    MongoDB shell version: 3.2.0
    connecting to: localhost:1111/test

Now, off to the races. Now I need to read all the collections, generate the hugo files, upload it to Github and would be done with it. So I thought, Scala, well too easy, let's do it Clojure way
using Mongoer but let's leave that to for a next post.
