# ts3serverARM
I managed to run some x86 applications like the Teamspeak (32 or 64 bit) server on my rpi 3B+ using a virtualized chroot environment (under the **64-bit** version of Ubuntu Server for Rpi).

## Tested on
- Raspberry Pi 3B+ 
- Raspberry Pi 3A+ (thanks to https://github.com/capsload2 !)

## Requirements
- Be sure to use the **4.0.0** (or later) version of `qemu`, `qemu-user`, and `qemu-user-static`.

If the 4.0.0 version isn't available, on Raspbian you can use Debian's `sid` repository && for Ubuntu use `eoan` or newer repos [you can change the repos by editing your /etc/apt/sources.list file; [anyway Google is your best friend](https://google.it) ]
- `debootstrap`
- `binfmt-support`
- `binutils`

## Creation of the x86 enviroment
- The `i386` architecture is 32-bit. The `amd64` architecture is 64-bit. Choose which one you like the most. x64 should be generally faster, mostly due to standardized register usage for function calls.
- I will be creating an `amd64` environment from now on.

1) Use *debootstrap* to download the *base system* in a folder called "./chroot-debian" (folder's name is up to you - in this case we'll be downloading **Debian stable**, you can use others as well): `sudo debootstrap --arch amd64 stable ./chroot-debian http://ftp.us.debian.org/debian`

2) Copy the qemu amd64 static emulator to the x86 environment's bin directory: `sudo cp /usr/bin/qemu-amd64-static ./chroot-stable/usr/bin/`
3) Mount the following directories to the x86 environment's directories (**REMEMBER** that you need to do this on **EVERY** system restart):
```
sudo mount -t sysfs sys ./chroot-debian/sys/
sudo mount -t proc proc ./chroot-debian/proc/
sudo mount --bind /dev ./chroot-debian/dev/
sudo mount --bind /dev/pts ./chroot-debian/dev/pts/
sudo mount --bind /dev/shm ./chroot-debian/dev/shm/
```

## Entering in the x86 environment
You can use chroot to enter the environment: `sudo chroot ./chroot-debian/`
 then `cd ~` to return to the default root directory.

## Creation of the ts3 server
Be aware that this environment acts as a parallel distro install from the host Raspberry's system, so for example, if you need to use a certain program, you need to reinstall it using `apt` (or whatever its installation method is) in the newly created environment.

- **FROM NOW WE ARE INSIDE THE x86 ENVIRONMENT, the commands might take A LOT to run because we are using an emulator inside the Raspberry whose specs aren't the best in the world. The `amd64` architecture should be faster than `i386`.** - 

If Apt asks you to install a package without verification, type YES.

1) Install `ca-certificates`, `bzip2` and `wget` packages: `apt install ca-certificates bzip2 wget`
2) Download the teamspeak3 (**32bit** or **64bit** version depending on your environment) server using `wget` (*short - depends on your net speed*): `wget https://files.teamspeak-services.com/releases/server/3.11.0/teamspeak3-server_linux_x86-3.11.0.tar.bz2` (you can find the latest versions [here](https://teamspeak.com/en/downloads/#server))
3) Untar the downloaded archive: `tar -xvf teamspeak3-server_linux_x86-3.11.0.tar.bz2`
4) Enter the folder: `cd teamspeak3-server_linux_x86`
5) Accept Teamspeak's license by using the file method: `touch .ts3server_license_accepted` (the touch command creates a file named .ts3server_license_accepted)

## *First run* of the server
1) run `./ts3server_minimal_runscript.sh` (**the ts3 server needs to compute a security puzzle at every start without signaling it on the console, so have patience**)
2) Copy the created Server Admin token and enter it when connecting to the server
3) yay, Connected! 


[well if you have problems open an issue]

## *Running* the server in background
**from the base system**
use `screen`: `apt install screen`

`sudo screen -dm sudo chroot ./chroot-stable/ /root/teamspeak3-server_linux_x86/ts3server_minimal_runscript.sh`

to kill a screen instance, [this is its documentation](https://www.gnu.org/software/screen/manual/screen.html).


## Running the server automatically on boot
To accomplish this I created a systemd service in the following way:

Create a new service file: `sudo nano /etc/systemd/system/teamspeak3.service`.

Copy the following text into the service file:
```
[Unit]
Description=Teamspeak Qemu Server
After=syslog.target network.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/home/gabboxl/chroot-stable/
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
~- When someone connects to the server, most probably it will spam `Unsupported ancillary data: 0/8` or similar text in the console. I didn't encounter any problem with server functionalities but at the time I didn't find a fix for that. [https://bugs.launchpad.net/qemu/+bug/1619896]~ [On QEMU 8.2.2 looks really stable - https://gitlab.com/qemu-project/qemu/-/issues/127]
- When trying to stop the server, you may encounter a segmentation fault error.

# TODO
- [ ] add images


# Credits
Created by **Gabboxl**

If this guide helped you in any way I ask you to **star** this project!
If you have encountered a problem open a new issue here on GitHub and I'll try to help you; do not hesitate!
