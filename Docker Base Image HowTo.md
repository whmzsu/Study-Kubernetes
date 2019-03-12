# How To Build Base image and where to download
## Ubuntu

Offical docker image download：https://hub.docker.com/_/ubuntu/

In general, start with a working machine that is running the distribution you’d like to package as a parent image, though that is not required for some tools like Debian’s Debootstrap, which you can also use to build Ubuntu images.

It can be as simple as this to create an Ubuntu parent image on a exsit Ubuntu system:

```bash
$ sudo apt install debootstrap
$ sudo mkdir ./xenial
$ sudo debootstrap --arch=amd64 xenial ./xenial > /dev/null
$ sudo tar -C xenial -c . |sudo  docker import - ubuntu-xenial
```

## Debian

Offical docker image download：https://hub.docker.com/_/debian

Same as Ubuntu, Debian docker base image can be built by the debootstrap command under a debian system.

```bash
$ sudo apt install debootstrap
$ sudo mkdir ./stable-chroot
$ sudo debootstrap --arch=amd64 stable ./stable-chroot http://deb.debian.org/debian/
$ sudo tar -C stable-chroot -c . |sudo  docker import - debian-stable
```

## RedHat Enterprise Linux
If you have subscription to the redhat product, the easiest way is downloading from  https://access.redhat.com/containers/#/explore, and choose the suitable version.

or build it by yourself

Build Procedure

1)download binary mkimage-yum.sh from https://github.com/docker/docker/blob/master/contrib/mkimage-yum.sh or git clone https://github.com/docker/docker.git

2)modify the mkimage-yum.sh to create a rhel 7 minimal tarfile, comment out the following two lines and add the third line as follows:

#tar –numeric-owner -c -C “$target” . | docker import - $name:$version

#docker run -i -t $name:$version echo success

tar –numeric-owner -c -C “$target” . -zf ${name}.tar.gz

3)use command “./mkimage-yum.sh rhel7_docker” to create the rhel7_docker.tar.gz tarfile.

4) copy the tarfile to another host where a docker daemon is running.

5) use command “cat rhel7_docker.tar.gz | sudo docker import - richxsl/rhel7” to create a docker rhel7 minimal image. You should change the variable YOUR_NAME and the YOUR_NAME is also your docker hub registration name.


## CentOS

Offical docker image download： https://hub.docker.com/_/centos/

The latest version of CentOS is 7, You need a existing CentOS 7 system for this task. You can use a vagrant environment if you want. Besides that, you have to exectute the steps below as root.

### Build OS root tree
```bash
# Create a folder for our new root structure
$ export centos_root='/centos_image/rootfs'
$ mkdir -p $centos_root
# initialize rpm database
$ rpm --root $centos_root --initdb
# download and install the centos-release package, it contains our repository sources
$ yum reinstall --downloadonly --downloaddir . centos-release
$ rpm --root $centos_root -ivh --nodeps centos-release*.rpm
$ rpm --root $centos_root --import  $centos_root/etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
# install yum without docs and install only the english language files during the process
$ yum -y --installroot=$centos_root --setopt=tsflags='nodocs' --setopt=override_install_langs=en_US.utf8 install yum
# configure yum to avoid installing of docs and other language files than english generally
$ sed -i "/distroverpkg=centos-release/a override_install_langs=en_US.utf8\ntsflags=nodocs" $centos_root/etc/yum.conf
# chroot to the environment and install some additional tools
$ cp /etc/resolv.conf $centos_root/etc
# mount the device tree, as its required by some programms
$ mount -o bind /dev $centos_root/dev
$ chroot $centos_root /bin/bash <<EOF
yum install -y procps-ng iputils
yum clean all
EOF
$ rm -f $centos_root/etc/resolv.conf
$ umount $centos_root/dev
```

### Build Docker base image
```bash
# install and enable docker
$ yum install -y docker
...
$ systemctl start docker
# create docker image
$ tar -C $centos_root -c . | docker import - centos
sha256:28843acfa85226385d2db0196db72465729db38088ea98751d4a5c3d4c85ccd
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              28843acfa852        11 seconds ago      287.7 MB
# run docker image
$ docker run --rm centos cat /etc/redhat-release
CentOS Linux release 7.2.1511 (Core)
```
Via usual docker ways you can now export the image or push it to some registry.
