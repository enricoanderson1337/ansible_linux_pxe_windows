# Ansible Linux PXE Server for Windows ISO Deployments

A pure Linux PXE Server, that provides a full unanttended Windows PXE Installation with customization features such as RunOnce-Script after first boot.

__Disclaimer__: Tested on Ubuntu Server 22.04. This was one of my first ansible projects and I would do many things differently today.

## Crashcourse for ansible

If you are a linux beginner and/or have never used Ansible before, follow the steps.
First of all you will need a Ubuntu 22.04 Server installation with a static IP, passwordless ssh login and passwordless sudo.

### Configure a static IP on the pxe-Server

Ansible works over ssh and therefor it would be nice for your pxe-server to have a static ip.
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


### Make a ssh host entry for your pxe-Server

Ansible works over ssh and therefor you should be able to login via `ssh <hostname>` instead of `ssh <username>@<IP>`. So make an entry to your `~/.ssh/config`.

```
# ~/.ssh/config
Host pxe-server                 
    Hostname <IP>
    User <username>
```

Test the config by entering `ssh pxe-server`. You should be asked for the password, which we will change for key-auth in the next step.

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

## Clone the repository

`git clone https://github.com/enricoanderson1337/ansible_linux_pxe_windows.git`

## Required ISO files

I sadly cannot provide you with the WAIK or Windows-ISOs in this repository.
The WAIK iso is absolutely required, you can download it [here](https://www.microsoft.com/en-us/download/details.aspx?id=5753). Put the file named `KB3AIK_EN.iso` into `roles/pxe/files/KB3AIK_EN.iso`

## Windows ISOs

For a Windows ISO to work out of the box with zero user input I provide you with `unattended.xml` files for:

| Windows Version | Language | md5sum | Donwload-Link |
|---|---|---|---|
| Windows 7 x64 | English | | [archive.org](https://archive.org/details/win7-pro-sp1-english) |
| Windows 7 x64 | German | | [archive.org](https://archive.org/download/Windows7UltimateSP1x64German) |
| Windows 10 x64 | English | | google it, microsoft provides links |
| Windows 10 x64 | German | | google it, microsoft provides links |
| Windows Server 2008 R2 x64 EVAL | English | | google it, microsoft provides links |
| Windows Server 2012 R2 x64 EVAL | English | | google it, microsoft provides links |
| Windows Server 2019 x64 EVAL | English | | google it, microsoft provides links |







