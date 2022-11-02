# Ansible Linux PXE Server for Windows ISO Deployments

A pure Linux PXE Server, that provides a full unanttended Windows PXE Installation with customization features such as RunOnce-Script after first boot.

__Disclaimer__: Tested on Ubuntu Server 22.04. This was one of my first ansible projects and I would do many things differently today.

## Requirements for ansible

If you are a linux beginner and/or have never used Ansible before, follow the steps.
First of all you will need a Ubuntu 22.04 Server installation with a static IP, passwordless ssh login and passwordless sudo.

### Configure a static IP on the pxe-Server

Edit the `/etc/netplan/00-installer-config.yaml` with sudo/root-privileges.

```
# /etc/netplan/00-installer-config.yaml 
network:
  ethernets:
    # This is my DHCP-based Internet adapter
    enp0s3:
      dhcp4: true
    # This is my static IP for the PXE-Network
    enp0s10:
      dhcp4: false
      addresses:
        - 192.168.56.12/24
  version: 2
```

After the edit apply the config with `netplan apply` and check the result with `ip a`.


### Make a ssh host entry for your ubuntu pxe-Server

To be able to log on via `ssh <hostname>` instead of `ssh <username>@<IP>` make an entry to your `~/.ssh/config`

```
# ~/.ssh/config
Host pxe-server                 
    Hostname <IP>
    User <username>
```

Test the config by entering `ssh pxe-server`. You should be asked for the password, which will exchange for key-auth in the next step.

### Generate a SSH-Key on your host system

If you dont have a SSH-Key you can generate one with the following command.

```
ssh-keygen -t ed25519
```

### Copy the ssh-key to your pxe-server

```
ssh-copy-id pxeserver
```

Test if you dont need to enter a password anymore to login via ssh by typing `ssh pxe-server`.

### Passwordless sudo

Edit the `/etc/sudoers` file by entering `visudo` or just edit the sudeors file like a madman by typing `sudo vim /etc/sudeors`.

```
# /etc/sudeors
# Add the line with the <username> after the %sudo

%sudo   ALL=(ALL:ALL) ALL
<username>    ALL=(ALL:ALL) NOPASSWD:ALL
```



