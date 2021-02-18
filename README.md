# ts3serverARM
I managed to run some x86 applications like the Teamspeak (32 bit) server on my rpi 3B+ using a virtualized chroot environment (under the **64-bit** version of Ubuntu Server for Rpi).

## Tested on
- Raspberry Pi 3B+ 
- Raspberry Pi 3A+ (thanks to https://github.com/capsload2 !)

## Requirements
- Be sure to use the **4.0.0** (or later) version of `qemu`, `qemu-user` and `qemu-user-static` 

if the 4.0.0 version isn't available, on Raspbian you can use debian's `sid` repository && for ubuntu use `eoan` or newer repos [you can change the repos by editing your /etc/apt/sources.list file; [anyway google is your best friend](https://google.it) ]
- `debootstrap`
- `binfmt-support`
- `binutils`

## Creation of the x86 enviroment
- Use *debootstrap* to download the *base system* in a folder called "./chroot-stretch" (folder's name is up to you - in this case we'll be downloading **debian stretch**, you can use others aswell): `sudo debootstrap --arch i386 stretch ./chroot-stretch http://ftp.us.debian.org/debian`

- Copy the qemu i386 static emulator to the x86 environment's bin directory: `sudo cp /usr/bin/qemu-i386-static ./chroot-stretch/usr/bin/`
- Mount the following directories to the x86 environment (**REMEMBER** that you need to do this on **EVERY** system restart):
```
sudo mount -t sysfs sys ./chroot-stretch/sys/
sudo mount -t proc proc ./chroot-stretch/proc/
sudo mount --bind /dev ./chroot-stretch/dev/
sudo mount --bind /dev/pts ./chroot-stretch/dev/pts/
sudo mount --bind /dev/shm ./chroot-stretch/dev/shm/
```

## Entering in the x86 environment
You can use chroot to enter in the environment: `sudo chroot ./chroot-stretch/`
 then `cd ~` to return in the default root's directory.

## Creation of the ts3 server
Be aware that this a completely detached environment from the host raspberry's system, so for example, if you need to use a certain program that you have installed on the host system, you need to reinstall it using `apt` (or whatever its installation method is) in the newly created environment.

- **FROM NOW WE ARE INSIDE THE x86 ENVIRONMENT, the commands might take A LOT to run because we are using an emulator inside the raspberry which its specs aren't the best in the world** - 

!) if apt asks you to install a package without verification type YES.

- Install `ca-certificates` package (*very long*): `apt install ca-certificates`
- Install `bzip2` package for the tar program (*short*): `apt install bzip2`
- Download the teamspeak3 (**32bit** version) server using `wget` (*short - depends by your net speed*): `wget https://files.teamspeak-services.com/releases/server/3.11.0/teamspeak3-server_linux_x86-3.11.0.tar.bz2` (you can find the latest 32-bit version [here](https://teamspeak.com/en/downloads/#server))
- Untar the downloaded archive: `tar -xvf teamspeak3-server_linux_x86-3.11.0.tar.bz2`
- `cd teamspeak3-server_linux_x86`
- accept teamspeak's license by using the file method: `touch .ts3server_license_accepted` (the touch command creates a file named .ts3server_license_accepted)

## *First run* of the server
- run `./ts3server_minimal_runscript.sh` (**every start of the ts3 server (on the rpi) will take A LOT because it will need to compute a puzzle, so have patience**)
- Copy the created token and enter it when connecting to the server with the ts3client
- yay, Connected!

## *Running* the server in background
**from the base system**
use `screen`: `apt install screen`

`sudo screen -dm sudo chroot ./chroot-stretch/ /root/teamspeak3-server_linux_x86/ts3server_minimal_runscript.sh`

to kill a screen instance, [this is its documentation](https://www.gnu.org/software/screen/manual/screen.html).


## Running the server automatically on boot
To accomplish this I created a systemd service in the following way:

Create a new service file: `sudo nano /etc/systemd/system/teamspeak3.service`.

Copy the following text in it:
```
[Unit]
Description=Teamspeak Qemu Server
After=syslog.target network.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/home/gabboxl/chroot-stretch/
ExecStartPre=-sudo mount -t sysfs sys ./sys/ ; -sudo mount -t proc proc ./proc/ ; -sudo mount --bind /dev ./dev/ ; -sudo mount --bind /dev/pts ./dev/pts/ ; -sudo mount --bind /dev/shm ./dev/shm/
ExecStart=chroot . /root/teamspeak3-server_linux_x86/ts3server_minimal_runscript.sh
ExecStop=chroot . /root/teamspeak3-server_linux_x86/ts3server_startscript.sh stop
ExecReload=chroot . /root/teamspeak3-server_linux_x86/ts3server_startscript.sh restart

[Install]
WantedBy=multi-user.target
```
Then **modify** `WorkingDirectory`'s value so that it points to the folder where the chroot environment is located.

Run `sudo systemctl enable teamspeak3` to enable the service, and `sudo systemctl start teamspeak3` to start the server.

You're done!

## Final considerations & known issues
- When someone connects to the server, most probably it will spam `Unsupported ancillary data: 0/8` or similar text in the console. I didn't encounter any problem with server functionalities but at the time I didn't find a fix for that. [https://bugs.launchpad.net/qemu/+bug/1619896]
- When trying to stop the server, you may encounter a segmentation fault error.

# TODO
- [ ] add images


# Credits
Created by **Gabboxl**

If this guide helped you in any way I ask you to **star** this project!
If you have encountered a problem open a new issue here on Github and I'll try to help you; do not hesitate!
