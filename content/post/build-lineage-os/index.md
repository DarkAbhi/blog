+++
author = "Abhishek AN"
title = "How to build Lineage OS for your device"
date = "2018-12-25"
summary = "A step-by-step guide on building LineageOS from source for your Android device."
tags = [
    "Andriod",
    "Lineage OS"
]
categories = [
    "Android"
]
+++

{{< youtube 52h7x3VXknI >}}


Follow this video to begin building custom ROMs for your own device instead of demanding developers to make different ROMs for you.

## Requirements
1. Linux System
2. Understanding Capability
3. Good Internet Connection

## Handy Terms
SUDO - SuperUser DO  
JDK - Java Development Kit  
mkdir - make directory  
brunch - combination of lunch and breakfast  

## Step-by-Step Build Guide

### Step 1: Update System Packages
First, ensure your system is up to date:

```bash
sudo apt-get update
```

### Step 2: Install Java Development Kit
LineageOS requires Java 8 to compile:

```bash
sudo apt-get install openjdk-8-jdk
```

### Step 3: Install Build Dependencies
Install all the required packages for building Android. This includes compilers, libraries, and build tools:

```bash
sudo apt-get install git-core gnupg flex bison gperf build-essential \
  zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
  lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
  libgl1-mesa-dev libxml2-utils xsltproc unzip
```

For Ubuntu 16.04 and newer, you'll also need these additional packages:

```bash
sudo apt install bc bison build-essential curl flex g++-multilib gcc-multilib \
  git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev \
  libesd0-dev liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev \
  libwxgtk3.0-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool \
  squashfs-tools xsltproc zip zlib1g-dev
```

### Step 4: Set Up Repo Tool
The `repo` tool is used to manage the Android source repositories.

Create a bin directory in your home folder:

```bash
mkdir ~/bin
```

Add the bin directory to your PATH:

```bash
PATH=~/bin:$PATH
```

Download and install the repo tool:

```bash
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

### Step 5: Configure Git
Set up your Git identity for commits:

```bash
git config --global user.name "<Your name>"
git config --global user.email "<your email ID>"
```

Replace `<Your name>` and `<your email ID>` with your actual name and email address.

### Step 6: Initialize LineageOS Source
Create a directory for the LineageOS source code:

```bash
mkdir los14
cd los14
```

Initialize the repository for LineageOS 14.1 (based on Android 7.1.2):

```bash
repo init -u https://github.com/LineageOS/android.git -b cm-14.1
```

### Step 7: Download the Source Code
Sync the repository to download all the source code. This will take a while depending on your internet connection:

```bash
repo sync
```

### Step 8: Build LineageOS
Set up the build environment:

```bash
. build/envsetup.sh
```

Start the build process for your device (in this example, x500):

```bash
brunch x500
```

Replace `x500` with your device's codename.

Visit Here For More Info and To Learn:
https://source.android.com/source/initializing

