# Ansible Linux PXE Server for Windows ISO Deployments

A pure Linux PXE Server, that provides a full unanttended Windows PXE Installation with customization features such as RunOnce-Script after first boot.

__Disclaimer__: Tested on Ubuntu Server 22.04. This was one of my first ansible projects and I would do many things differently today.

# Requirements for Ansible

If you have never used Ansible before, follow the steps:

First of all you will need a Ubuntu 22.04 Server installation and a passwordless ssh login and passwordless sudo.

## Make a ssh host entry for your ubuntu pxe server

To be able to log on via `ssh <hostname>` instead of `ssh <username>@<IP>` make an entry to your `~/.ssh/config`

```
# ~/.ssh/config
Host pxe-server                 
    Hostname <IP>
    User <username>
```

Test the config by entering `ssh pxe-server`. You should be asked for the password, which will exchange for key-auth in the next step.

## Generate a SSH-Key on your host system

If you dont have a SSH-Key you can generate one with the following command.

```
ssh-keygen -t ed25519
```

## Copy the ssh-key to your pxe-server

```
ssh-copy-id pxeserver
```

Test if you dont need to enter a password anymore to login via ssh by typing `ssh pxe-server`.



