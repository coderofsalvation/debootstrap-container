debootstrap-container
=====================

<img src=".res/logo.png"/>

simple way of running multiple debian containers on a (openvz) VPS 

### Howto

    $ sudo debootstrap-container add mycontainer
    add ssh user? (y/n) y
    enter ssh username: myusername
    Enter new UNIX password: 
    Retype new UNIX password: 
    passwd: password updated successfully
    [x] done..you can login thru ssh using: ssh myusername@localhost
    enter user which should be 'root'? myusername
    [x] created container '/srv/containers/flop'
    $ ssh myusername@localhost
    root@mycontainer# whoami
    root@mycontainer# root
    root@mycontainer# apt-get install python2
    root@mycontainer# exit

now the funpart is: python2 is only installed in the container.
removing /srv/containers means all packages in the container are removed as well.

If you are only interested in local development, you can easily access the container locally as well:

    $ whoami 
    myusername
    $ debootstrap-container run /srv/containers/foo
    root@mycontainer# whoami
    root@mycontainer# root

Or how about exporting our container to a tarball

    $ debootstrap-container export /src/containers/mycontainer /tmp/mycontainer.tar.bz2
    [x] analyzing additional installed packages + backing up dirs: srv etc opt
    [x] written /tmp/mycontainer.tar.gz

Only the newly installed packagenames + some dirs are in the tar (instead of the whole rootfilesystem).
Lets overwrite our container with an exported tarball:

    $ debootstrap-container import /src/containers/mycontainer /tmp/mycontainer.tar.bz2  # overwrites container
    [x] Reading package lists...
    [x] Building dependency tree...
    [x] The following extra packages will be installed:
    [x] python2 

Or simple start a fresh container 'mycontainer2' based on our exported tarball:

    $ debootstrap-container import /src/containers/mycontainer2 /tmp/mycontainer.tar.bz2 # creates clone
    [x] Reading package lists...
    [x] Building dependency tree...
    [x] The following extra packages will be installed:
    [x] python2 
    [x] done
    $ ssh mycontainer2@localhost
    root@mycontainer# whoami (..and so on..)

### Why

I am a huge fan of [docker](http://docker.io) but unfortunately I had to do multiple projects inside one VPS server.
I could not get docker working, and solutions like [sekexe](https://github.com/jpetazzo/sekexe) or jailkit pointed into more possible hassle.
I had to go another road to satisfy my needs:

* I want to install packages in a container, *outside* the real host
* I want to easily import, export, delete containers
* I want to run node- or apache/lighttpd applications in a container
* I want to ssh to a container and feel like I have root-access
* I want to be somewhat compatible with docker
* I want dont want to waste diskspace (bare debootstrap container is 124mb)

### Docker compatibility

It *should* be compatible with docker, just tar your jail-dir like so:

    tar -C /srv/containers/mycontainer -c . | docker import myname/mycontainer

### Conclusion

This is definately not secure or as cool as docker, but it is an tidy way to deploy
applications on a openvz container (which doesnt run docker).
