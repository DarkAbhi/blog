+++
author = "Abhishek AN"
title = "How to find CAF tag of Kernel without git history"
date = "2019-01-02"
summary = "Learn how to identify the correct CAF kernel tag even when git history is unavailable"
tags = [
    "Linux",
    "Kernel",
    "Git"
]
categories = [
    "Linux"
]
+++

It is always best to have a clean and tidy git history for the linux kernel.

To find the tag that was used by your OEM and further modified as per the device needs. Firstly, grab the zip or tar kernel source for your device and extract it.

Then initialize a git repository in the kernel source folder. 
```html
git init
```
Then add the remote from CAF for your respective kernel version.

## For 3.18:
> https://source.codeaurora.org/quic/la/kernel/msm-3.18/

## For 4.4:

> https://source.codeaurora.org/quic/la/kernel/msm-4.4

## For 4.9:

> https://source.codeaurora.org/quic/la/kernel/msm-4.9

## Example 
```html
git remote add msm https://source.codeaurora.org/quic/la/kernel/msm-4.4
```

And fetch all the tags or fetch the tags you feel relevant. To fetch all tags type 
```html
git fetch msm 'refs/tags/*:refs/tags/*'
```

Commit in your kernel folder with a message
```html
git add .
git commit
```

Then download the script to check for the best CAF tag from [here](https://github.com/LineageOS/scripts/blob/master/best-caf-kernel/best-caf-kernel.py). And place it in your kernel folder. Then run the script using
```html
./best-caf-kernel.py "*-8x98.0"
```

As you can see above, this will check for all the tags ending with -8x98.0 for a msm8998 device. Assuming we found the best tag to be LA.UM.6.4.r1-05700-8x98.0

You should next fetch the tag and checkout to it and give it a new branch name
```html
git fetch https://source.codeaurora.org/quic/la/kernel/msm-4.4.git LA.UM.6.4.r1-05700-8x98.0
git checkout FETCH_HEAD
```

To verify you fetched it correctly type in **git log** and check the commit history. Then push the branch to your git repository and copy your kernel changes over the current CAF tag.
