# Vagrant workshop environment

This directory contains code to set up a Vagrant environment for running the Lightbulb workshop. This should work regardless the operating system you're running.

Beforehand, ensure the following software is installed

- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Vagrant](https://www.vagrantup.com/downloads.html)
- (optionally) [Git](https://git-scm.com/downloads)

You need at least the `Vagrantfile` and the `bootstrap.sh` in this directory. Download them and put them together in a directory on your system.

After that, open a shell and run the commands:

```console
> vagrant up
[ ... ]
> vagrant ssh
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
