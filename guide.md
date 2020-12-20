# Create your own image for jetson nano board

## Introduction

\_Note: This guide is copied from [https://pythops.com/post/create-your-own-image-for-jetson-nano-board](https://pythops.com/post/create-your-own-image-for-jetson-nano-board) so that it is still readable in the event that site goes down.
I made a few minor alterations to the guide to make it display right in markdown, and to fix some issues I had with the
guide.

This article will guide you step by step to create a minimalist Ubuntu based image for your NVIDIA Jetson nano board that best suits your project.

## Why would you build an image from scratch instead of using the official one ?

First, for fun. It's always interesting to build stuff from scratch as you always learn something in the process. Second, the official image is large in size (over 5GB 😱) and it's filled with lot of unnecessary preinstalled packages (ubuntu-desktop, browser …) that takes lot of disk space and memory.
So, let's create a clean and minimalist image.

## Download the scripts

Before starting, let's first clone the repository where I put all the needed scripts.

```bash
$ git clone https://github.com/pythops/jetson-nano-image
$ cd jetson-nano-image
```

## Create a rootfs

We're gonna use the script `create-rootfs.sh` to create a basic rootfs.
First, we define the location where we want to build it. This is done by defining the environment variable `$JETSON_ROOTFS_DIR`. The path will be created if it does not exit.

```bash
$ export JETSON_ROOTFS_DIR=/path/to/rootfs
```

Then we build the rootfs by running the following command

```bash
$ sudo -E ./create-rootfs.sh
ROOTFS_DIR: ~/jetson-rootfs
Installing the dependencies...  [OK]
Creating rootfs directory...    [OK]
Downloading the base image...   [OK]
Run debootstrap first stage...  [OK]
Run debootstrap second stage... [OK]
Success!
```

_Note: -E option for sudo preserve the environment variables_

## Customize

Now that we have a basic rootfs, we're gonna customize it using one of my favorite tool ever: `Ansible`.
For this step you need to have Ansible installed in your workstation. if it's not the case, just run this command and you're ready to go

```bash
$ sudo apt install ansible
```

This Ansible role is going to do 3 things:

- Install some basic tools (ssh, systemd, sudo …)
- Setup basic configurations (locales, network …)
- Add new user: pythops

You can run the playbook as follows

```bash
$ cd ansible
$ sudo -E $(which ansible-playbook) jetson.yaml
```

Feel free to adapt this role to your needs.

## Create the image

Now that we customized the rootfs, we're gonna use the script `create-image.sh` to create our final image.
We need to define a build directory using the environment variable `$JETSON_BUILD_DIR`. This path will be created if it does not exist.

```bash
$ export JETSON_BUILD_DIR=/path/to/build_dir
```

Then we build the image as follows

```bash
$ sudo -E ./create-image.sh
```

If all goes well, you'll get this message at the end

```bash
Image created successfully
Image location: /path/to/jetson.img
```

## Flash on the sdcard

Finally we're gonna flash the image on the sdcard using the script `flash-image.sh`.

Insert your sdcard and run this command

```bash
$ sudo ./flash-image.sh /path/to/jetson.img /dev/mmcblk0
.
.
.
Success !
Your sdcard is ready !
```

_Note: the sdcard path /dev/mmcblk0 may be different in your system._

This script will automatically resize the root partition to occupy all the available space in the sdcard.

Congratulations 🎉 Now you can boot your board with the new image !

## Nvidia Libraries

To install nvidia libraries (cuda, libcudnn, tensorRT …) you need to use [nvidia sdk manager](https://developer.nvidia.com/nvidia-sdk-manager).

## Result

With the new image only 200MB of RAM is used, which leaves you with 3.8 GB for your projects !
