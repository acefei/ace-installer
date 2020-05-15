# ace-osinstaller
To use iPXE to setup my own development machine

This repo is used for creating `ipxe.iso` without deploying extra DHCP server/tFTP server.

## Getting Started

### Prerequisites
Promptly start as long as there is `docker` (curl -fsSL https://get.docker.com | sh) and `make` on your host and get help running `make` in the root path of this repo.

### Usage
#### Use official ipxe.iso
1. Generate boot.ipxe and put it on http server using `make http_server`
2. Download official [ipxe.iso](http://boot.ipxe.org/ipxe.iso) and follow the steps on [Quick Start](https://ipxe.org/) into PXE cli 
3. Run the following cmd
```
iPXE> dhcp
iPXE> chain http://<your http server ip>/boot.ipxe
```

#### Build your own ipxe.iso
Preferred chain loading that we don't build `ipxe.iso` everytime unless HTTP_SERVER changed and only need to update `boot.ipxe` on the fly. 
```
make http_server HTTP_SERVER=<the ip for fetching boot.ipxe over HTTP>
```
> There are two artifacts, `output/ipxe.iso` and `www/boot.ipxe` and it will launch a http server against www/ dir 
Then, burn ipxe.iso onto a blank CD-ROM or DVD-ROM or put it into the ISO library for the VM installation on XenServer/Vmware/KVM

For more details usage, just run `make` to get help.

### Add Extra Distro
You can find available distro configuration in [gen_embedded.json](https://github.com/acefei/ace-osinstaller/blob/master/scripts/gen_embedded.json)
 
1. Add new section as below for new distro support 
```
"New Distro Name": {
        "description": "the details for distro",
        "url": "http://<your local server ip>",
        "kernel": "<relative path>/vmlinuz",
        "initrd": "<relative path>/initrd.img",
        "kernel_args": "<literal as the key name>"
    },
```
2. Put new distro iso into [www](https://github.com/acefei/ace-osinstaller/tree/master/www) dir
3. Restart http server `make http_server`
4. If wanted to tweak the kernel args, we only need to change it in `www/boot.ipxe` instead of building `ipxe.iso` again because of chain loading feature.

<details>
  <summary>There are something specials in answerfile</summary>

1. Use /dev/xvda which is simply the Xen disk storage devices as disk partition , you need to update it if you use other Hypervisor
2. Use Text mode instead of desktop environment
3. Create an encrypted password for the user configuration in answerfile
 ```
   python3 -c 'import crypt,getpass;pw=getpass.getpass();print(crypt.crypt(pw) if (pw==getpass.getpass("Confirm: ")) else exit())'
 ```
4. Install [ace-profile](https://github.com/acefei/ace-profile) in the post install stage
</details>

## How to dump answerfile infomation after OS installed
### Centos
- cat /root/anaconda-ks.cfg
### Debian/Ubuntu
1. cat /var/log/installer/cdebconf/questions.dat
2. install `debconf-utils` and run `debconf-get-selections -installer` to dump preseed

## Acknowledgments

* [iPXE Download](http://ipxe.org/download)
* [The system initialization(Debian)](https://www.debian.org/doc/manuals/debian-reference/ch03.en.html) 
* [Error building ISO](https://forum.ipxe.org/showthread.php?tid=8080)
* [Debian (stable) preseed example](https://www.debian.org/releases/stable/example-preseed.txt)
* [Ubuntu (stable) preseed example](https://help.ubuntu.com/stable/installation-guide/example-preseed.txt)
* [Automated Server Installs for 20.04](https://wiki.ubuntu.com/FoundationsTeam/AutomatedServerInstalls#Differences_from_debian-installer_preseeding)
* [How to unpack/uncompress and repack/re-compress an initial ramdisk (initrd/initramfs) boot image file](https://access.redhat.com/solutions/24029)
  * If ran into the err `No TPM chip found, activating TPM-bypass!`, please run `find . 2>/dev/null | cpio -o -H newc | xz -9 --format=lzma > /tmp/initrd.img` to repack, instead of the one in the link above.

## Known Issues
* Failed to retrieve preconfiguration file ubuntu 1804 as wget in busybox can not download https url without `--no-check-certificate`, that need to wait busybox upgrade.
* Load ipxe.iso error as below, it works perfectly after installing `isolinux` instead of `syslinux`. 
```
Boot device: CD-Rom0MB medium detected
- failure: could not read boot disk
```
