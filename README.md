As for 21-02-2019, what I can see on the internet is, that atom was successfully built by a github user "hypersad" who shared the method to do so in his repository. Unfortunately that user no longer exist on github and so his respositories. I have tried to find (looks like his) resources on a different repo (thanks to the owner of that repo) which I will share below. The build process did not perfect for me, but I was able to successfully install and then use atom on raspi 3B without errors.

The detailed method to install atom on armv7l architectures (like raspi3B) can be seen here (apparently from the original author):

https://libraries.io/github/hypersad/atom-armv7l

# atom-armv7l (original method, can be skipped to "my method section")
👋 Hello! This repository contains several patched files and instructions that allow you to build and run Atom 1.23 on armv7l machines like Raspberry Pi's.

## Building
Gettings sources & building
Firstly, clone Atom sources from 1.23-releases branch and this repository, then merge them:
```
git clone https://github.com/atom/atom.git -b 1.23-releases
git clone https://github.com/hypersad/atom-armv7l.git
cp -r atom-armv7l/script/* atom/script/
rm -rf atom-armv7l
```
Now you could start building:

`cd atom
script/build`
Building Atom on armv7l machines usually takes around 1-2 hours, but build time depends on your hardware.

## Generating startup blob separately
This steps must be done on i386/amd64 machine.

We will use mksnapshot binaries for i386/amd64 with armv7l as target to get startup blob. I've tried to build mksnapshot binaries for armv7l that works natively, but Atom just throws Illegal instruction error with their blobs. Currently, I don't know how to skip this step and make snapshot natively. You could try to use i386 emulator for this, if you want of coure.

## Get suitable mksnapshot binary. For now it is mksnapshot-v1.6.0-linux-armv7l:

```
wget https://github.com/electron/electron/releases/download/v1.6.0/mksnapshot-v1.6.0-linux-armv7l.zip
unzip -j mksnapshot-v1.6.0-linux-armv7l.zip mksnapshot
rm mksnapshot-v1.6.0-linux-armv7l.zip
```
Get generated startup snapshot (out/startup.js) from your armv7l machine. You could do it via scp, for example:

`scp helix@192.168.1.30:/home/helix/atom/out/startup.js startup.js`
Generate startup blob from startup.js via mksnapshot:

`./mksnapshot startup.js --startup_blob snapshot_blob.bin`
Send generated startup blob back to your armv7l machine. You could do it via scp too, for example:

`scp snapshot_blob.bin helix@192.168.1.30:/home/helix/atom/out/atom-1.23.2-armv7l/snapshot_blob.bin`
Now, you could try to launch Atom! 🎉

# My-Method (some modifications)

Start with the following:

## Prerequisites
armv7l machine. You can check it by typing `uname -m` to terminal emulator.

Node.js 6.10.2. Highly recommended to install it via Node.js version manager. Installation guide can be found here.

`nvm install 6.10.2` - install Node.js 6.10.2.

`nvm alias default 6.10.2` - set Node.js 6.10.2 as system default Node.js version.

`nvm use 6.10.2` - set Node.js version to 6.10.2 for this session.

Node.js package manager.
`sudo apt-get install npm` - install Node.js package manager.

`sudo npm install -g npm` - update Node.js package manager.

`npm config set python /usr/bin/python2` - locate Python 2 path for Node-GYP.

Git.
`sudo apt-get install git`

C++ 11 Toolchain, development headers for GNOME Keyring, and other build tools.
`sudo apt-get install build-essential git libgnome-keyring-dev fakeroot rpm libx11-dev libxkbfile-dev`
```
git clone https://github.com/atom/atom.git -b 1.23-releases
git clone https://github.com/osamazhar/atom-armv7l-1.git (originally cloned from ricehiroko, thanks to him)
cp -r atom-armv7l/script/* atom/script/
rm -rf atom-armv7l
cd atom
```

Before you start building you will have to remove some packages as they may not be found by atom build script. For me, they are as follows:
```
github
language-c
language-javascript
```
Remove them from `packages.json` from the root folder.

Now build the sources:

`script/build`

After all the packages are installed, you may stuck at the memory leak issue when the compiler installs atomdocs. So wait for the error to come and use the following commit to change the corresponding file (the location of the file will be displayed on the error message)

`https://github.com/atom/atomdoc/commit/60197a43cd87342bebf027c138bb46d270644a76`

Now you can pass this error also. You will notice that the packaging of the app will not be finished and the build process will be stopped by passing an error. I have no solution to solve this error, but you will notice that there is an out folder in the root directory and you can go to `atom/out/atom-1.23.3-armv7l/` and you should be able to run `./atom` from this folder.

You may still face an package manager error if you try to install packages from atom. That error is because `apm` is not copied in out folder. So you can do so manually.

Take the `apm` folder from the root of source and copy it to `out/atom-1.23.3-armv7l/resources/app` folder

Restart atom and you should be able to install any package from atom.

Now you will manually have to copy the built files to the system location. I chose this to be `/opt`

Go to `/opt` and make a new directory mkdir atom

Now copy all the contents of `/out/atom-1.23.3-armv7l` to `/opt/atom`

Don't forget to change the permissions of the binaries. `sudo chmod a+rx /path/to/binaries`

I changed permission of atom (in `/opt/atom` and `apm`, `node` and `npm` in `/opt/atom/resources/app/apm/node_modules/atom-package-manager/bin`)

Now the last step is to create symlinks of these to `/usr/local/bin` folder. See if you already have these files in that folder, if yes (and the version is the same as that of those you used to build atom) then don't make any symlink. create a symlink of atom from `/opt/atom/atom`.

For a desktop icon, you can create one like this:

create a file in `~/Desktop` as `Atom.desktop` and add the details as follows:

```
[Desktop Entry]
Version=1.0
Type=Application
Terminal=false
Exec=command to run here
Name=visible name here
Comment=comment here
Icon=icon path here
```
Icon can be found at `/opt/atom/resources/app.asar.unpacked/resources/atom.png`

And now you should have a working atom in your raspi 3B (or possibly on other armv7l systems).

