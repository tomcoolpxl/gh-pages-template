# Set-up

## Clone a VM server

### Prerequisites

- [VMware](https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html)
- [ISO Linux](https://ubuntu.com/download/server)
- [Ubuntu Server Install - preparation](https://hogeschoolpxl-my.sharepoint.com/:w:/g/personal/20003821_pxl_be/EcqwJVS9NCBFsgNMgiQ-bsABbI3L0phbzL-e7oOdaGjflA?e=Gry2ol)

### Clone VM - Start with a pre-installed vm

ubuntu-server-1 - 192.168.206.131/24

power off the main vm before making a clone

``` bash
sudo poweroff
```

### Create a full clone

#### Create a clone of an existing virtual machine

![VMWare clone](images/vwware_clone.jpg)

### Altering the Clone

#### Change the name of the host

The `/etc/hostname` is the file to register hostnames in the Linux operating systems. It contains the configuration for all local hostnames. You can edit the file and change the computer name as per your needs.

Check the content of the hostname.

``` bash
cat /etc/hostname
```

``` bash
sudo sed -i 's/server-1/server-2/g' /etc/hostname
```

-i: It stands for "in-place," and when used with sed, it indicates that the file should be edited in-place, meaning the changes will be made directly to the file, and no backup or temporary files will be created.

or use

``` bash
sudo hostnamectl set-hostname ubuntu-server-2
```

Check if the hostname is altered.

``` bash
cat /etc/hostname
```

Unfortunately, the `hostnamectl` command doesn’t update `/etc/hosts` for you, so you’ll need to edit that file yourself to make the error 'unable to resolve host' go away.

If the hostname mapping of 127.0.0.1 does not match the hostname, the sudo command will take a very long time due to name-resolving timeout.

### Change hosts file

``` bash
sudo sed -i 's/server-1/server-2/g' /etc/hosts
```

``` bash
cat /etc/hosts
```

#### Poor man's dns

- The `/etc/hosts` file on is a static lookup table that maps IP addresses to hostnames
- Has precedence over, and performs the same functionality as, DNS
- Useful for fast testing in local networks
- You can add entries manually
- 127.0.0.1 must always be mapped to the current local hostname

![Alt text](image-2.png)

### Change machine-id

Change random number by changing some characters in the string. The virtual machine will receive a new IP address from the DHCP server.

``` bash
sudo nano /etc/machine-id
```

Reboot the server

``` bash
sudo reboot
```

### Renew the SSH Server host-keys

The easiest way to do this, is by reinstalling the openssh-server.

``` bash
sudo apt -y purge openssh-server && sudo apt -y install openssh-server
```

Location of the SSH host-keys

``` bash
sudo ls /etc/ssh/*key*
```

- Check if the hostname is correct in the new generated SSH public keys.

<!-- SOLUTION
cat *.pub 
-->

Reboot the server

``` bash
sudo reboot
```

The new VM gets a different IP address from the VMWare DHCP server and has a new name. For example 'ubuntu-server-2'

### Test the new clone

#### Check the IP

The IP address of this server 2 should be different from the IP of server 1.

``` bash
ip a
```

#### Try to ping the new clone

For example:

- ubuntu-server-1 - 192.168.206.131/24
- ubuntu-server-2 - 192.168.206.132/24

##### server-1

``` bash
ping 192.168.206.132/24
```

##### server-2

``` bash
ping 192.168.206.131/24
```

### VMWare NAT config

Edit -> Virtual Network Editor

![Alt text](image-3.png)

![Alt text](image-4.png)

VMnet0 -> Bridged

![Alt text](vm_auto_bridge.png)

VMnet8 -> NAT

![Alt text](vm_nat.png)

Windows Terminal

``` bash
Get-NetIPConfiguration *VMnet8
```

<!-- netsh interface show interface | findstr /I VMnet* -->

```console
InterfaceAlias       : VMware Network Adapter VMnet8
InterfaceIndex       : 5
InterfaceDescription : VMware Virtual Ethernet Adapter for VMnet8
IPv4Address          : 192.168.206.1
IPv6DefaultGateway   :
IPv4DefaultGateway   :
DNSServer            : fec0:0:0:ffff::1
                       fec0:0:0:ffff::2
                       fec0:0:0:ffff::3
```

### Add a Disk to VM

- Go to the setting of your virtual machine, right-click on you vm.

![Alt text](image-5.png)

- Press -> Settings

![Alt text](image-6.png)

- Press the add button

![Alt text](image-7.png)

- Select Hard Disk and press the Next button

![Alt text](image-8.png)

- Select SCSI and press the Next button

![Alt text](image-9.png)

- Select "Create a new virtual Disk" and press the Next button

![Alt text](image-10.png)

- Choose the maximum disk size and, split disk into multiple files and press the Next button.

![Alt text](image-11.png)

Press Finish -> Press Ok

#### Reboot the virtual machine

``` bash
sudo reboot
```

## Time Zone
### Check Time Zone

``` bash
timedatectl
```

List the available Time Zones

``` bash
timedatectl list-timezones
```

Search of a specific Time Zone

``` bash
timedatectl list-timezones | grep -i brussels
```

Print the system date and time

``` bash
date
```

### Change Time Zone

Set a specific Time Zone

``` bash
sudo timedatectl set-timezone Europe/Brussels
```

``` bash
whoami
```

## Add a user

Add a user student2 on your Ubuntu client (Server1) with password 'pxl'

``` bash
 sudo adduser student2
 ```

 ``` bash
[sudo] password for student:
Adding user `student2' ...
Adding new group `student2' (1001) ...
Adding new user `student2' (1001) with group `student2' ...
Creating home directory `/home/student2' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for student2
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
```

[How-to: Create a user on ubuntu](https://phoenixnap.com/kb/how-to-create-sudo-user-on-ubuntu)

### Add user student2 to the sudo group

``` bash
sudo usermod -aG sudo student2
```

``` bash
[sudo] password for student:
```

The `-aG` option tells the system to append the user to the specified group. (The -a option is only used with G.)

### Verify User Belongs to sudo Group

``` bash
groups student2
```
