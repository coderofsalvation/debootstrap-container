debootstrap-container
=====================

<img src=".res/logo.png"/>

simple way of running multiple debian ssh-able containers on a (openvz) VPS 

> CAUTION: this shellscript requires root, use at own risk.

# Usage

    Usage: 
    
    debootstrap-container delete <containername> [containerpath]
    debootstrap-container add <containername> [release] [variant] [containerpath] 
    debootstrap-container run <containerdir/name> 
    debootstrap-container showreleases

### Howto

    $ sudo debootstrap-container add foo
    enter user which should be 'root'? foo 
    adding new user 'foo'
    Enter new UNIX password: 
    Retype new UNIX password: 
    passwd: password updated successfully 
    redirect user into container upon ssh-login? (y/n) y
    [x] created container '/srv/containers/foo'

    $ ssh foo@localhost
    foo@foo# whoami
    uid=1003 gid=1004 groups=1004

    foo@foo# sudo whoami
    root

    foo@foo# sudo apt-get install python2
    foo@foo# exit

now the funpart is: python2 is only installed in the container, and the sudo is 
actually fakeroot.

### Why

I am a huge fan of [docker](http://docker.io) but unfortunately I had to do multiple projects inside one VPS server.
I could not get docker working, and solutions like [sekexe](https://github.com/jpetazzo/sekexe) or jailkit pointed into more possible hassle.
I had to go another road to satisfy my needs:

* I want to install packages in a container, *outside* the real host
* I want to easily backup/restore containers
* I want to run node- or apache/lighttpd applications in a container
* I want to ssh to a container and feel like I have root-access
* I want to be somewhat compatible with docker
* Doesnt use too much diskspace (bare debootstrap container is 124mb)

### Docker compatibility

It *should* be compatible with docker, just tar your jail-dir like so:

    tar -C /srv/containers/mycontainer -c . | docker import myname/mycontainer

### TIP: shared directories

Sharing directories across containers (originating from outside the container) can be handy.
However, avoid symbolic links since it will confuse applications when resolving absolute paths, instead mount like so:

    mount --bind /opt/somefolder /srv/containers/mycontainer/opt/somefolder

> WARNING: always use 'debootstrap-container delete <yourcontainer>' to delete a container.
Using a plain 'sudo rm -rf /srv/containers/yourcontainer' might cause dataloss for folders using 'mount --bind'

### TIP: persistent containers

To keep the container alive after logouts/timeouts:

* get a passwordless ssh-login working:
 
 
    $ su foo
    foo $ ssh-keygen <enter><enter><enter>
    foo $ ssh-copy-id foo@localhost
    foo $ exit
    $ ln -fs /srv/containers/foo/home/foo/.ssh /home/foo/.
 
 
* run gnu 'screen' as root (apt-get install screen)
* inside this screen, ssh into your container(s) (ssh foo@localhost)
* and leave it there (ctrl A-D)

By doing so, the master-screen process will always be persistent, and you can just ssh from anywhere directly into your ssh-container. You can even run a screen inside your container if you want to.
The following files would automate this:

/root/.screen 

    screen -T xterm -t container_foo 1 su foo -c 'ssh foo@localhost'
    screen -T xterm -t container_bar 2 su bar -c 'ssh bar@localhost'

/etc/init.d/screen 

    #!/bin/bash
    find /srv/containers -mindepth 1 -maxdepth 1 -type d | while read dir; do       
      /root/lemon/server/debootstrap-container/debootstrap-container mount_dirs "$dir"
    done
    echo /usr/bin/flock -w 0 /root/.screen.lock /usr/bin/screen -s /bin/bash -d -m &

This would mount some needed folders (/sys /proc) and start+detach the screen during startup.

### Sudo 

The containers use a sudo-shellscript at /usr/bin/sudo which uses fakeroot.
To prevent users from installing software please remove the /usr/bin/fakeroot binary.

### History

* Sun Apr 26 21:43:47 CEST 2015 refactor: debootstrap doesnt run by root-user, but with sudo cmds
* Sun Apr 26 21:43:47 CEST 2015 refactor: added remove function 
* Mon Apr 27 12:53:50 CEST 2015 refactor: removed import/export: just tar the container at /src/container/*

### Conclusion

This is definately not secure or as cool as docker, but it is an tidy way to deploy
applications on a openvz container (which doesnt run docker).
