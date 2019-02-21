As for 21-02-2019, what I can see on the internet is, that atom was successfully built by a github user "hypersad" who shared the method to do so in his repository. Unfortunately that user no longer exist on github and so his respositories. I have tried to find (looks like his) resources on a different repo (thanks to the owner of that repo) which I will share below. The build process did not perfect for me, but I was able to successfully install and then use atom on raspi 3B without errors.

The detailed method to install atom on armv7l architectures (like raspi3B) can be seen here (apparently from the original author):

https://libraries.io/github/hypersad/atom-armv7l

# atom-armv7l
👋 Hello! This repository contains several patched files and instructions that allow you to build and run Atom 1.23 on armv7l machines like Raspberry Pi's.

## Building
Gettings sources & building
Firstly, clone Atom sources from 1.23-releases branch and this repository, then merge them:

git clone https://github.com/atom/atom.git -b 1.23-releases
git clone https://github.com/hypersad/atom-armv7l.git
cp -r atom-armv7l/script/* atom/script/
rm -rf atom-armv7l
Now you could start building:

cd atom
script/build
Building Atom on armv7l machines usually takes around 1-2 hours, but build time depends on your hardware.

## Generating startup blob separately
This steps must be done on i386/amd64 machine.

We will use mksnapshot binaries for i386/amd64 with armv7l as target to get startup blob. I've tried to build mksnapshot binaries for armv7l that works natively, but Atom just throws Illegal instruction error with their blobs. Currently, I don't know how to skip this step and make snapshot natively. You could try to use i386 emulator for this, if you want of coure.

## Get suitable mksnapshot binary. For now it is mksnapshot-v1.6.0-linux-armv7l:

wget https://github.com/electron/electron/releases/download/v1.6.0/mksnapshot-v1.6.0-linux-armv7l.zip
unzip -j mksnapshot-v1.6.0-linux-armv7l.zip mksnapshot
rm mksnapshot-v1.6.0-linux-armv7l.zip
Get generated startup snapshot (out/startup.js) from your armv7l machine. You could do it via scp, for example:

scp helix@192.168.1.30:/home/helix/atom/out/startup.js startup.js
Generate startup blob from startup.js via mksnapshot:

./mksnapshot startup.js --startup_blob snapshot_blob.bin
Send generated startup blob back to your armv7l machine. You could do it via scp too, for example:

scp snapshot_blob.bin helix@192.168.1.30:/home/helix/atom/out/atom-1.23.2-armv7l/snapshot_blob.bin
Now, you could try to launch Atom! 🎉
