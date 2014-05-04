debootstrap-container
=====================

<img src=".res/logo.png"/>

simple way of running multiple debian containers on a (openvz) VPS instead of docker 

### Why

I am a huge fan of [docker](http://docker.io) but unfortunately I had to do multiple projects inside one VPS server.
I could not get docker working, and solutions like [sekexe](https://github.com/jpetazzo/sekexe) pointed into more possible hassle.
I had to go another road to satisfy my needs:

* I want to install packages in a container, *outside* the real host
* I want to easily import, export, delete containers
* I want to run node- or apache/lighttpd applications in a container
* I want to ssh to a container and feel like I have root-access
* I want to be somewhat compatible with docker

### Howto

    $ debootstrap-container add mycontainer
    [x] created container in /srv/containers
    [x] adding user
    [x] done..now login using ssh with login/pass: lemon/lemon
    $ ssh mycontainer@localhost
    root@mycontainer# whoami
    root@mycontainer# root
    root@mycontainer# passwd (enter my newpassword)
    root@mycontainer# apt-get install python2
    root@mycontainer# exit

now the funpart is that python2 is only installed in the container.
removing /srv/containers means all packages in the container are removed as well.

    $ debootstrap-container export /src/containers/mycontainer /tmp/mycontainer.tar.bz2
    [x] analyzing additional installed packages + backing up dirs: srv etc opt
    [x] written /tmp/lemon.tar.gz

We now exported our container to a tarball.

    $ debootstrap-container import /src/containers/mycontainer /tmp/mycontainer.tar.bz2  # overwrites container

We just updated our container with a tarball

    $ debootstrap-container import /src/containers/mycontainer2 /tmp/mycontainer.tar.bz2 # creates clone

We just updated imported our container to a new tarball
    
    $ ssh mycontainer2@localhost
    root@mycontainer# whoami (..and so on..)


### Docker compatibility

It *should* be compatible with docker, just tar your jail-dir like so:

    tar -C /srv/containers/mycontainer -c . | docker import myname/mycontainer

### Conclusion

This is definately not secure or as cool as docker, but it is an tidy way to deploy
applications on a openvz container (which doesnt run docker).
