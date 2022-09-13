# Managing Kernels

## Generic Linux Information

Unfortunately a real problem with security is managing the kernel versions that collide with security antivirus solutions. 

To find your Linux release (printed as bash vars):

`$ cat /etc/*-release`

```
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu 22.04.1 LTS"
(snip)
```

LSB (alternative):

`$ lsb_release -a`

```
LSB Version:    core-5.0-amd64:core-5.0-noarch
Distributor ID: openSUSE project
Description:    openSUSE Leap 42.2
Release:        42.2
Codename:       n/a
```

To show your complete ***current running*** kernel version string:

`$ uname -a`

```
Linux my-host-name 5.15.0-43-generic #46-Ubuntu SMP Tue Jul 12 10:30:17 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```

To show only the ***current running*** kernel version number:

`$ uname -r`
 
```
 5.15.0-43-generic
```

To list GRUB available bootable kernels installed:

`$ find /boot/config-*`

```
/boot/config-5.15.0-43-generic
/boot/config-5.15.0-47-generic
```


## CrowdStrike Falcon Sensor

Fetch and download the latest sensor.. (put it in your network as CS requires OATH and multiple API calls to fetch the deb package)

```shell
$ wget http://192.168.86.33:5000/falcon-sensor_6.45.0-14203_amd64.deb
```

You can extract `falcon-kernel-check` as single file from the deb using `dpkg & tar`

```shell
$ dpkg --fsys-tarfile falcon-sensor_6.45.0-14203_amd64.deb \
| tar xOf - ./opt/CrowdStrike/falcon-kernel-check14203 \
> falcon-kernel-check
```

### falcon-kernel-check

This lets you run a bash script (that has no external deps)

```shell
$ ./falcon-kernel-check
Host OS 5.4.0-125-generic #141-Ubuntu SMP Wed Aug 10 13:42:03 UTC 2022 is supported by Sensor version 14203.
```

Additional options support `-k` and proper shell returns 

```
$ ./falcon-kernel-check -k 1.1.1
1.1.1 is not supported by Sensor version 14203.
$ echo $?
1
```
## Ubuntu 

Get a list of installed kernels via `dpkg`

`$ dpkg -l |grep -e 'linux\-\(image\|header\).*'`

```
ii  linux-headers-5.15.0-43               5.15.0-43.46                            all          Header files related to Linux kernel version 5.15.0
ii  linux-headers-5.15.0-43-generic       5.15.0-43.46                            amd64        Linux kernel headers for version 5.15.0 on 64 bit x86 SMP
ii  linux-headers-5.15.0-47               5.15.0-47.51                            all          Header files related to Linux kernel version 5.15.0
ii  linux-headers-5.15.0-47-generic       5.15.0-47.51                            amd64        Linux kernel headers for version 5.15.0 on 64 bit x86 SMP
ii  linux-headers-generic                 5.15.0.47.47                            amd64        Generic Linux kernel headers
ii  linux-image-5.15.0-43-generic         5.15.0-43.46                            amd64        Signed kernel image generic
ii  linux-image-5.15.0-47-generic         5.15.0-47.51                            amd64        Signed kernel image generic
ii  linux-image-generic                   5.15.0.47.47                            amd64        Generic Linux kernel image
```

### Lock Kernels 

If you boot with the `linux-image-4.13.0-50-generic` kernel version and run `apt autoremove` then if `linux-image-4.13.0-26-generic` is also installed it will be purged if it's not the most recent or the next most recent version installed. Additionally autoremove will never remove the current running kernel. 

`sudo apt autoremove`, will not target applications using either apt-mark or dpkg hold

#### Method 1 - `apt-mark`
Set the hold to prevent the current kernel from getting updated or purged:

`$ sudo apt-mark hold linux-image-generic linux-headers-generic`
`$ sudo apt-mark hold linux-image-$(uname -r) linux-headers-$(uname -r)`

To remove the hold:

`$ sudo apt-mark unhold linux-image-generic linux-headers-generic`
`$ sudo apt-mark unhold linux-image-4.13.0-26-generic`

#### Method 2 - `dpkg`

Set the hold to prevent the kernel from getting purged:

```
$ echo linux-image-4.13.0-26-generic hold | dpkg --set-selections
$ echo linux-headers-4.13.0-26-generic hold | dpkg --set-selections
```
To remove the hold:

```
$ echo linux-image-4.13.0-26-generic install | dpkg --set-selections
$ echo linux-headers-4.13.0-26-generic install | dpkg --set-selections
```

#### Method 3 - (Only stop unattended)

Stop the Ubuntu Kernel Update via config file:

`sudo vi /etc/apt/apt.conf.d/50unattended-upgrades`

Scroll down and locate the blacklist section and edit like below with the Linux kernel packages. Here wildcards are also supported.

```json
Unattended-Upgrade::Package-Blacklist {
"linux-generic";
"linux-image-generic";
"linux-headers-generic";
};
```

#### Validate the Hold

```
$ dpkg -l |grep -e 'linux\-\(image\|header\).*'
ii  linux-headers-5.4.0-125               5.4.0-125.141                      all          Header files related to Linux kernel version 5.4.0
hi  linux-headers-5.4.0-125-generic       5.4.0-125.141                      amd64        Linux kernel headers for version 5.4.0 on 64 bit x86 SMP
hi  linux-headers-generic                 5.4.0.125.126                      amd64        Generic Linux kernel headers
hi  linux-image-5.4.0-125-generic         5.4.0-125.141                      amd64        Signed kernel image generic
hi  linux-image-generic                   5.4.0.125.126                      amd64        Generic Linux kernel image
```

### Listing All Available Kernels via APT

Get the list of all the available Kernels via `apt`:

```
$ sudo apt update
$ apt list linux-*image-*
```

For a more **complete list available **sorted:

You can find the dates here: http://security.ubuntu.com/ubuntu/pool/main/l/linux-signed/

```
$ apt list linux-*image-* |grep generic | grep "linux-image-[4-9].*" |sort -V
```

```
WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

linux-image-5.4.0-26-generic/focal 5.4.0-26.30 amd64
linux-image-5.4.0-28-generic/focal-updates,focal-security 5.4.0-28.32 amd64
linux-image-5.4.0-29-generic/focal-updates,focal-security 5.4.0-29.33 amd64
linux-image-5.4.0-31-generic/focal-updates,focal-security 5.4.0-31.35 amd64
linux-image-5.4.0-33-generic/focal-updates,focal-security 5.4.0-33.37 amd64
linux-image-5.4.0-37-generic/focal-updates,focal-security 5.4.0-37.41 amd64
linux-image-5.4.0-39-generic/focal-updates,focal-security 5.4.0-39.43 amd64
linux-image-5.4.0-40-generic/focal-updates,focal-security 5.4.0-40.44 amd64
linux-image-5.4.0-42-generic/focal-updates,focal-security 5.4.0-42.46 amd64
linux-image-5.4.0-45-generic/focal-updates,focal-security 5.4.0-45.49 amd64
linux-image-5.4.0-47-generic/focal-updates,focal-security 5.4.0-47.51 amd64
linux-image-5.4.0-48-generic/focal-updates,focal-security 5.4.0-48.52 amd64
linux-image-5.4.0-51-generic/focal-updates,focal-security 5.4.0-51.56 amd64
linux-image-5.4.0-52-generic/focal-updates,focal-security 5.4.0-52.57 amd64
linux-image-5.4.0-53-generic/focal-updates,focal-security 5.4.0-53.59 amd64
linux-image-5.4.0-54-generic/focal-updates 5.4.0-54.60 amd64
linux-image-5.4.0-58-generic/focal-updates,focal-security 5.4.0-58.64 amd64
linux-image-5.4.0-59-generic/focal-updates,focal-security 5.4.0-59.65 amd64
linux-image-5.4.0-60-generic/focal-updates,focal-security 5.4.0-60.67 amd64
linux-image-5.4.0-62-generic/focal-updates,focal-security 5.4.0-62.70 amd64
linux-image-5.4.0-64-generic/focal-updates 5.4.0-64.72 amd64
linux-image-5.4.0-65-generic/focal-updates,focal-security 5.4.0-65.73 amd64
linux-image-5.4.0-66-generic/focal-updates,focal-security 5.4.0-66.74 amd64
linux-image-5.4.0-67-generic/focal-updates,focal-security 5.4.0-67.75 amd64
linux-image-5.4.0-70-generic/focal-updates,focal-security 5.4.0-70.78 amd64
linux-image-5.4.0-71-generic/focal-updates,focal-security 5.4.0-71.79 amd64
linux-image-5.4.0-72-generic/focal-updates,focal-security 5.4.0-72.80 amd64
linux-image-5.4.0-73-generic/focal-updates,focal-security 5.4.0-73.82 amd64
linux-image-5.4.0-74-generic/focal-updates,focal-security 5.4.0-74.83 amd64
linux-image-5.4.0-77-generic/focal-updates,focal-security 5.4.0-77.86 amd64
linux-image-5.4.0-80-generic/focal-updates,focal-security 5.4.0-80.90 amd64
linux-image-5.4.0-81-generic/focal-updates,focal-security 5.4.0-81.91 amd64
linux-image-5.4.0-84-generic/focal-updates,focal-security 5.4.0-84.94 amd64
linux-image-5.4.0-86-generic/focal-updates,focal-security 5.4.0-86.97 amd64
linux-image-5.4.0-88-generic/focal-updates,focal-security 5.4.0-88.99 amd64
linux-image-5.4.0-89-generic/focal-updates,focal-security 5.4.0-89.100 amd64
linux-image-5.4.0-90-generic/focal-updates,focal-security 5.4.0-90.101 amd64
linux-image-5.4.0-91-generic/focal-updates,focal-security 5.4.0-91.102 amd64
linux-image-5.4.0-92-generic/focal-updates,focal-security 5.4.0-92.103 amd64
linux-image-5.4.0-94-generic/focal-updates,focal-security 5.4.0-94.106 amd64
linux-image-5.4.0-96-generic/focal-updates,focal-security 5.4.0-96.109 amd64
linux-image-5.4.0-97-generic/focal-updates,focal-security 5.4.0-97.110 amd64
linux-image-5.4.0-99-generic/focal-updates,focal-security 5.4.0-99.112 amd64
linux-image-5.4.0-100-generic/focal-updates,focal-security 5.4.0-100.113 amd64
linux-image-5.4.0-104-generic/focal-updates,focal-security 5.4.0-104.118 amd64
linux-image-5.4.0-105-generic/focal-updates,focal-security 5.4.0-105.119 amd64
linux-image-5.4.0-107-generic/focal-updates,focal-security 5.4.0-107.121 amd64
linux-image-5.4.0-109-generic/focal-updates,focal-security 5.4.0-109.123 amd64
linux-image-5.4.0-110-generic/focal-updates,focal-security 5.4.0-110.124 amd64
linux-image-5.4.0-113-generic/focal-updates,focal-security 5.4.0-113.127 amd64
linux-image-5.4.0-117-generic/focal-updates,focal-security 5.4.0-117.132 amd64
linux-image-5.4.0-120-generic/focal-updates,focal-security 5.4.0-120.136 amd64
linux-image-5.4.0-121-generic/focal-updates,focal-security 5.4.0-121.137 amd64
linux-image-5.4.0-122-generic/focal-updates,focal-security 5.4.0-122.138 amd64
linux-image-5.4.0-124-generic/focal-updates,focal-security 5.4.0-124.140 amd64
linux-image-5.4.0-125-generic/focal-updates,focal-security,now 5.4.0-125.141 amd64 [installed,automatic]
linux-image-5.8.0-23-generic/focal-updates 5.8.0-23.24~20.04.1 amd64
linux-image-5.8.0-25-generic/focal-updates 5.8.0-25.26~20.04.1 amd64
linux-image-5.8.0-28-generic/focal-updates 5.8.0-28.30~20.04.1 amd64
linux-image-5.8.0-29-generic/focal-updates 5.8.0-29.31~20.04.1 amd64
linux-image-5.8.0-33-generic/focal-updates,focal-security 5.8.0-33.36~20.04.1 amd64
linux-image-5.8.0-34-generic/focal-updates,focal-security 5.8.0-34.37~20.04.2 amd64
linux-image-5.8.0-36-generic/focal-updates,focal-security 5.8.0-36.40~20.04.1 amd64
linux-image-5.8.0-38-generic/focal-updates,focal-security 5.8.0-38.43~20.04.1 amd64
linux-image-5.8.0-40-generic/focal-updates 5.8.0-40.45~20.04.1 amd64
linux-image-5.8.0-41-generic/focal-updates,focal-security 5.8.0-41.46~20.04.1 amd64
linux-image-5.8.0-43-generic/focal-updates,focal-security 5.8.0-43.49~20.04.1 amd64
linux-image-5.8.0-44-generic/focal-updates,focal-security 5.8.0-44.50~20.04.1 amd64
linux-image-5.8.0-45-generic/focal-updates,focal-security 5.8.0-45.51~20.04.1+1 amd64
linux-image-5.8.0-48-generic/focal-updates,focal-security 5.8.0-48.54~20.04.1 amd64
linux-image-5.8.0-49-generic/focal-updates,focal-security 5.8.0-49.55~20.04.1 amd64
linux-image-5.8.0-50-generic/focal-updates,focal-security 5.8.0-50.56~20.04.1 amd64
linux-image-5.8.0-53-generic/focal-updates,focal-security 5.8.0-53.60~20.04.1 amd64
linux-image-5.8.0-55-generic/focal-updates,focal-security 5.8.0-55.62~20.04.1 amd64
linux-image-5.8.0-59-generic/focal-updates,focal-security 5.8.0-59.66~20.04.1 amd64
linux-image-5.8.0-63-generic/focal-updates,focal-security 5.8.0-63.71~20.04.1 amd64
linux-image-5.11.0-22-generic/focal-updates,focal-security 5.11.0-22.23~20.04.1 amd64
linux-image-5.11.0-25-generic/focal-updates,focal-security 5.11.0-25.27~20.04.1 amd64
linux-image-5.11.0-27-generic/focal-updates,focal-security 5.11.0-27.29~20.04.1 amd64
linux-image-5.11.0-34-generic/focal-updates,focal-security 5.11.0-34.36~20.04.1 amd64
linux-image-5.11.0-36-generic/focal-updates,focal-security 5.11.0-36.40~20.04.1 amd64
linux-image-5.11.0-37-generic/focal-updates,focal-security 5.11.0-37.41~20.04.2 amd64
linux-image-5.11.0-38-generic/focal-updates,focal-security 5.11.0-38.42~20.04.1 amd64
linux-image-5.11.0-40-generic/focal-updates,focal-security 5.11.0-40.44~20.04.2 amd64
linux-image-5.11.0-41-generic/focal-updates,focal-security 5.11.0-41.45~20.04.1 amd64
linux-image-5.11.0-43-generic/focal-updates,focal-security 5.11.0-43.47~20.04.2 amd64
linux-image-5.11.0-44-generic/focal-updates,focal-security 5.11.0-44.48~20.04.2 amd64
linux-image-5.11.0-46-generic/focal-updates,focal-security 5.11.0-46.51~20.04.1 amd64
linux-image-5.13.0-21-generic/focal-updates,focal-security 5.13.0-21.21~20.04.1 amd64
linux-image-5.13.0-22-generic/focal-updates,focal-security 5.13.0-22.22~20.04.1 amd64
linux-image-5.13.0-23-generic/focal-updates,focal-security 5.13.0-23.23~20.04.2 amd64
linux-image-5.13.0-25-generic/focal-updates,focal-security 5.13.0-25.26~20.04.1 amd64
linux-image-5.13.0-27-generic/focal-updates,focal-security 5.13.0-27.29~20.04.1 amd64
linux-image-5.13.0-28-generic/focal-updates,focal-security 5.13.0-28.31~20.04.1 amd64
linux-image-5.13.0-30-generic/focal-updates,focal-security 5.13.0-30.33~20.04.1 amd64
linux-image-5.13.0-35-generic/focal-updates,focal-security 5.13.0-35.40~20.04.1 amd64
linux-image-5.13.0-37-generic/focal-updates,focal-security 5.13.0-37.42~20.04.1 amd64
linux-image-5.13.0-39-generic/focal-updates,focal-security 5.13.0-39.44~20.04.1 amd64
linux-image-5.13.0-40-generic/focal-updates,focal-security 5.13.0-40.45~20.04.1 amd64
linux-image-5.13.0-41-generic/focal-updates,focal-security 5.13.0-41.46~20.04.1 amd64
linux-image-5.13.0-44-generic/focal-updates,focal-security 5.13.0-44.49~20.04.1 amd64
linux-image-5.13.0-48-generic/focal-updates,focal-security 5.13.0-48.54~20.04.1 amd64
linux-image-5.13.0-51-generic/focal-updates,focal-security 5.13.0-51.58~20.04.1 amd64
linux-image-5.13.0-52-generic/focal-updates,focal-security 5.13.0-52.59~20.04.1 amd64
linux-image-5.15.0-33-generic/focal-updates,focal-security 5.15.0-33.34~20.04.1 amd64
linux-image-5.15.0-41-generic/focal-updates,focal-security 5.15.0-41.44~20.04.1 amd64
linux-image-5.15.0-43-generic/focal-updates,focal-security 5.15.0-43.46~20.04.1 amd64
linux-image-5.15.0-46-generic/focal-updates,focal-security 5.15.0-46.49~20.04.1 amd64
```

### Deleting old Kernels

List all kernels available for possible deletion

```
$ dpkg -l 'linux-image-*' | sed '/^ii/!d;/'"$(uname -r | sed "s/\(.*\)-\([^0-9]\+\)/\1/")"'/d;s/^[^ ]* [^ ]* \([^ ]*\).*/\1/;/[0-9]/!d'
```

Purge the old Kernels

```
$ sudo apt-get remove --purge $(dpkg -l 'linux-image-*' | sed '/^ii/!d;/'"$(uname -r | sed "s/\(.*\)-\([^0-9]\+\)/\1/")"'/d;s/^[^ ]* [^ ]* \([^ ]*\).*/\1/;/[0-9]/!d')
```

Update the Grub Bootloader

`$ update-grub` 

## RedHat

Red Hat Enterprise Linux Release Dates

* https://access.redhat.com/articles/3078

Remember `yum list` shows available packages, while `yum list installed` & `rpm -qa` shows only those which were installed.

### Lock Kernels 

Yum and DNF support to versionlock

#### Method 1 (preferred): via `versionlock`

Reference: https://access.redhat.com/solutions/98873

To exclude kernels from being upgraded via YUM update:

```
$ yum install -y yum-plugin-versionlock
(..snip..)
Installed:
  yum-plugin-versionlock.noarch 0:1.1.31-42.el7 
```

> For RHEL 5
> `$ yum install yum-versionlock`
> 
> For RHEL 6 and 7
> `$ yum install yum-plugin-versionlock`
> 
> For RHEL 8 and 9
> `$ yum install python3-dnf-plugin-versionlock`

Enable the plugin `/etc/yum/pluginconf.d/versionlock.conf`:

```ini
[main]
enabled = 1
locklist = /etc/yum/pluginconf.d/versionlock.list
#  Uncomment this to lock out "upgrade via. obsoletes" etc. (slower)
# follow_obsoletes = 1
```

To add an application to the lock:

`$ yum versionlock kernel-*`

Edit the locklist:

`$ vi /etc/yum/pluginconf.d/versionlock.list `

```
kernel-3.10.0-693.2.2.el7
```

Automatically add the kernel to the list:

``$ sudo sh -c 'rpm -qa | grep kernel-* >> /etc/yum/pluginconf.d/versionlock.list'``

To display the list of locked packages, use:

`$ yum versionlock list`

To discard the list of locked packages, use:

`$ yum versionlock clear`


#### Method # 2: `yum –exclude` command to lock package version from yum update

Edit /etc/yum.conf

`$ vi /etc/yum.conf`

Append the following line under `[main]` section to lock kernel enter:

`exclude=kernel* `

### List Kernels 

Using `yum` to list available kernels

`
$ yum info 'kernel*' -q
`

```
  Available Packages
  Name        : kernel
  Arch        : x86_64
  Version     : 3.10.0
  Release     : 693.11.6.el7
  Size        : 43 M
  Repo        : updates/7/x86_64
  Summary     : The Linux kernel
  URL         : http://www.kernel.org/
  License     : GPLv2
  Description : The kernel package contains the Linux kernel (vmlinuz), the core of any
              : Linux operating system.  The kernel handles the basic functions
              : of the operating system: memory allocation, process allocation, device
              : input and output, etc.
```

To check all installed kernels:
```
$ yum list kernel -q
  Installed Packages
  kernel.x86_64            3.10.0-693.11.1.el7           @updates
  kernel.x86_64            3.10.0-693.11.6.el7           @updates
```

To check kernel packages available for upgrade:

```
$ yum check-update kernel*
Loaded plugins: fastestmirror, langpacks
base                                               | 3.6 kB  00:00:00     
extras                                             | 3.4 kB  00:00:00     
updates                                            | 3.4 kB  00:00:00     
Loading mirror speeds from cached hostfile
 * base: ftp.iitm.ac.in
 * extras: ftp.iitm.ac.in
 * updates: ftp.iitm.ac.in

kernel.x86_64                    3.10.0-693.2.2.el7               updates
kernel-tools.x86_64              3.10.0-693.2.2.el7               updates
kernel-tools-libs.x86_64         3.10.0-693.2.2.el7               updates
```

### Upgrade Kernels

To upgrade the kernel, you can run this yum command:

`$ yum upgrade kernel -y`


### Deleting old Kernels

To remove old installed kernels:

> Using the command package-cleanup with the `--oldkernels` switch would remove all old kernels, leaving only `--count` most recent ones (by default count=2). For example, to remove all kernels except the one most recently installed and loaded, run the following command

`package-cleanup --oldkernels --count=1`


