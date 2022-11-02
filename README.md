![logo](https://github.com/enricoanderson1337/ansible_linux_pxe_windows/blob/main/media/logo.png))

A pure Linux PXE Server, that provides a full unanttended Windows PXE Installation with customization features such as RunOnce-Script after first boot. Tested on Ubuntu Server 22.04.

__Disclaimer__: 
- WARNING: The PXE-Entries will automatically format your 1st disk, so be warned!
- This was one of my first ansible projects and I would do many things differently today.
- Give your pxe-server some disk space, usually 2.5 time the combined iso-sizes (80GB in my case).

## ansible crashcourse

If you are a linux beginner or have never used ansible before, follow these steps.
First of all you will need a ubuntu server 22.04 installation with a static IP, passwordless ssh login and passwordless sudo.
I wont explain how to install ubuntu server 22.04. I personally use VirtualBox, give it 80GB of disk space and two network adapters, first one NAT and second one Host-Only network for pxe deployments.

![preview](https://github.com/enricoanderson1337/ansible_linux_pxe_windows/blob/main/media/video.gif)

### Configure a static IP on the pxe-Server

Ansible works over ssh and therefore it would be nice for your pxe-server to have a static ip.
Edit the `/etc/netplan/00-installer-config.yaml` with sudo/root-privileges.

```
# /etc/netplan/00-installer-config.yaml 
network:
  ethernets:
    # This is my dhcp-based internet adapter
    enp0s3:
      dhcp4: true
    # This is my static IP for the pxe-Network
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
The WAIK iso is required, you can download it [here](https://www.microsoft.com/en-us/download/details.aspx?id=5753). Put the file named `KB3AIK_EN.iso` into `roles/pxe/files/KB3AIK_EN.iso`

## Windows ISOs

For a Windows ISO to work out of the box with zero user input I provide you with `unattended.xml` files for the following versions:

| Windows Version | Language | md5sum | Download-Link | Preconfigured path and filename |
|---|---|---|---|---|
| Windows 7 x64 | English | | [archive.org](https://archive.org/details/win7-pro-sp1-english) | roles/pxe/files/Windows_7_x64_eng.iso |
| Windows 7 x64 | German | | [archive.org](https://archive.org/download/Windows7UltimateSP1x64German) | roles/pxe/files/Windows_7_x64_ger.iso |
| Windows 10 x64 | English | | google it, microsoft provides links | roles/pxe/files/Windows_10_x64_eng.iso |
| Windows 10 x64 | German | | google it, microsoft provides links | roles/pxe/files/Windows_10_x64_ger.iso |
| Windows Server 2008 R2 x64 EVAL | English | | google it, microsoft provides links | roles/pxe/files/Windows_Server_2008R2_eng_eval.iso |
| Windows Server 2012 R2 x64 EVAL | English | | google it, microsoft provides links | roles/pxe/files/Windows_Server_2012R2_eng_eval.iso |
| Windows Server 2019 x64 EVAL | English | | google it, microsoft provides links | roles/pxe/files/Windows_Server_2019_eng_eval.iso |

## Edit the ansible group_vars

Depending on the isos you downloaded, edit and/or change the entries.
If you need other entries, then you need to create your own `unattended.xml` files.

```
---
windows:
  - name: windows_server_2008r2_eng
    bootentry: Windows Server 2008 R2 English
    bootcategory: server
    isofile: Windows_Server_2008R2_eng_eval.iso
    unattendedfile: windows_server_2008r2_unattend.xml.j2
    bootstrapscript: windows_all_bootstrap.cmd.j2
    startscript: windows_server_2008r2_startscript.cmd.j2
[...]

# Unimportant
pxeserver:
  # Options: normal.png | rarepepe.png | evilpooh.png
  background: normal.png
  directory: /pxeboot
  directories:
    - /pxebuild
    - /pxebuild/waikiso
    - /pxebuild/winpeiso
    - /pxebuild/winpeiso_modified
    - /pxebuild/winpeiso_wim

pxebuild:
  directory: /pxebuild
  waikisomountdir: /pxebuild/waikiso
  winpeisomountdir: /pxebuild/winpeiso
  winpemoddir: /pxebuild/winpeiso_modified
  winpewimmountdir: /pxebuild/winpeiso_wim

samba:
  directory: /pxesamba
  isomountdir: /pxesamba/iso
```

## Edit the ansible inventory

The pxe-server needs some config info from you, so put your details in the `inventory/hosts.yaml` file.

```
# Group name, leave it as it is
pxe_servers:
  hosts:
    # The actual hostname of your server, change the value if you named it differently in your ssh-config/dns
    pxe-server:
      network_config:
        # the interface name, where the pxe services should listen on
        interface_name: "enp0s10"
        # the ip of the interface
        interface_ip: "192.168.56.12"
        dhcp_range:
          # the start and end of the dhcp range, assuming /24 range
          start: "192.168.56.230"
          end: "192.168.56.240"
```

## Run the playbook

The process can take several minutes, because the iso-files will be copied and extracted on the pxe-server.

`ansible-playbook -i inventory/hosts.yaml playbook_bootstrap_pxe.yaml`

## Test the result

Start a client in the same network and choose network-boot. you should boot into the pxe-menu.
Username is usually `admin:admin` and `Administrator:admin`.

