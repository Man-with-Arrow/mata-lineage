# mata-lineage
All you need to build LineageOS yourself for the Essential PH-1.

This is a WIP, and currently fails for me with brillo not being able to create an OTA properly. The exact error is:

```
[ 99% 98333/98334] Package OTA: /srv/src/LINEAGE_15_1/out/target/product/mata/lineage_mata-ota-2790ed3123.zip
FAILED: /srv/src/LINEAGE_15_1/out/target/product/mata/lineage_mata-ota-2790ed3123.zip 
```

After that failure, brillo tries to continue with making the images, but it hits Broken pipe because of this line in platform_system_update_engine:

```
# Brillo images are zip files. We detect the 4-byte magic header of the zip
# file.
local magic=$(head --bytes=4 "${image}" | hexdump -e '1/1 "%.2x"')
```


I'm putting this here so it'll be easier to replicate my process, and find out what I'm doing wrong. 


## What do I need?

1. Dependencies
2. Docker
3. Docker image

## Installing dependencies
Using your package manager of choice, install the following:

`bc bison build-essential curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-gplv2-dev lib32z1-dev libesd0-dev liblz4-tool libncurses5-dev libsdl1.2-dev libwxgtk2.8-dev libxml2 libxml2-utils lzop pngcrush schedtool squashfs-tools xsltproc zip zlib1g-dev`

Copy the `manifests` folder in this repo to your `home` folder. To clarify - this means that the contents of `/home/$you/manifests` should be `mata.xml` and `additional_packages.xml`.


## Install Docker

The official Docker guides are well-written:

&nbsp; 

* Linux ([Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/), [Debian](https://docs.docker.com/install/linux/docker-ce/debian/), [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/) and [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/) are officially supported)
* [Windows 10/Windows Server 2016 64bit](https://docs.docker.com/docker-for-windows/install/)
* [Mac OS El Capitan 10.11 or newer](https://docs.docker.com/docker-for-mac/install/)

If your Windows or Mac system doesn't satisfy the requirements (or if you have Oracle VirtualBox installed) you can use [Docker Toolbox](https://docs.docker.com/toolbox/overview/).

Docker Toolbox is not described in this guide, but it should be very similar to the standard Docker installation.

Once you can run the [`hello-world` image](https://docs.docker.com/get-started/#test-docker-installation) you're ready to start!

## Use the Docker image 

First-time builders, run the following in a terminal, **substituting `user` with your username!**
If you've built before, substitute the default paths for the ones you already have, and have configured. 

&nbsp;

```
docker run \ 
-e "BRANCH_NAME=lineage-15.1" \ 
-e "DEVICE_LIST=mata" \ 
-e "SIGN_BUILDS=true" \ 
-e "SIGNATURE_SPOOFING=restricted" \ 
-e "CUSTOM_PACKAGES=GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension " \ 
-e "INCLUDE_PROPRIETARY=false" \ 
-e "CCACHE_SIZE=20G" \
-v "/home/user/lineage:/srv/src" \ 
-v "/home/user/zips:/srv/zips" \ 
-v "/home/user/logs:/srv/logs" \ 
-v "/home/user/cache:/srv/ccache" \ 
-v "/home/user/keys:/srv/keys" \ 
-v "/home/user/manifests:/srv/local_manifests" \ 
lineageos4microg/docker-lineage-cicd
```

If you want it pre-rooted, add in `-e "WITH_SU=true" `. 