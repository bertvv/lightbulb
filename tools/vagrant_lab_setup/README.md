# Vagrant workshop environment

This directory contains code to set up a Vagrant environment for running the Lightbulb workshop. This should work regardless the operating system you're running.

Beforehand, ensure the following software is installed

- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Vagrant](https://www.vagrantup.com/downloads.html)
- (optionally) [Git](https://git-scm.com/downloads)

You only need the `Vagrantfile` in this directory. Download it and preferrably put it in a separate directory.

After that, open a shell and run the commands:

```console
$ vagrant up
[ ... ]
$ vagrant ssh
[vagrant@ansible ~]$ ansible all -m ping
ansible | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node-1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
[ ... ]
```
