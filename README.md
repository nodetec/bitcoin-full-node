# Bitcoin Full Node Setup

You will need:

- Raspberry Pi 4 (4GB Memory, at least 32GB micro SD storage)
- Official Rpi Power brick
- Ethernet cord (optional but recommended)
- Minimum 1TB external drive (for storing the blockchain)
- Non metered internet connection

[Link parts needed](https://www.amazon.com/hz/wishlist/ls/1FSSY7WAANYXG?ref_=wl_share)

## OS

We need an OS, and the best one that I could think of is the official OS developed by the people who make the Raspberry Pi.

[Raspberry Pi OS Docs](https://www.raspberrypi.com/documentation/computers/getting-started.html) 
[Raspberry Pi Imager](https://www.raspberrypi.com/software/) is the recommended way to install Raspberry Pi OS. 

```
sudo apt install rpi-imager
```

I recommend installing the `Raspberry Pi OS (64-bit)` for now you can find that in the *other* section.

When installing you will also have the option to configure a few settings when clicking the gear in the bottom right.

I recommend at least checking the box to automatically enable ssh.

Make sure you have your SD card inserted into the computer, and when you're ready click the write button and wait for the OS to install.

## Interacting with the Pi

You can use the Pi same as any other computer, so if you want to just plugin a mouse, keyboard and monitor that's an option. If you do things this way you will have access to graphical various tools.

Another option is to ssh into the Pi from another computer. Which is what I will be doing and this tutorial will assume you are doing everything via SSH. You can obviously still easily follow along with your graphical interface by just opening up a terminal.

## Format external Drive

To find the drive enter the following:

```
sudo fdisk -l
```

You should see something like:

```
Device     Start        End    Sectors  Size Type
/dev/sda1   2048 3907028991 3907026944  1.8T Linux filesystem
```

You will know it's the right drive because the Size should be much larger than the micro SD card. Also take note of `/dev/sda1/` this tells us that we will be targeting `/dev/sda` when we go to format that drive.

We can now begin formatting the drive using `fdisk`:

```
sudo fdisk /dev/sda
```

You can press `m` to get help understanding the different operations available.

We will do the following:

- `g`: create a new empty GPT partition table

- `n`: add a new partition (press `Enter` to proceed with using the whole drive)

- `y`: confirm and remove signature (NOTE this may not be necessary if there is no signature to remove)

- `w`: write table to disk and exit

Running `sudo fdisk -l` will show your new partition

Now we can build our file system with:

```
sudo mkfs.exfat /dev/sda1
```

## Mounting USB External Drive

We will now need to mount the drive and also make sure it automatically mounts when we reboot. We'll also change the permissions to allow the user read, write and execute access.

**NOTE** change `pi` to your username


```
sudo mkdir /mnt/bitcoin

sudo chown -R pi:pi /mnt/bitcoin # change pi to your username

sudo chmod -R 775 /mnt/bitcoin

sudo mount /dev/sda1 /mnt/bitcoin
```

Set all future permissions for the mount point to pi user and group:

```
sudo setfacl -Rdm g:pi:rwx /mnt/bitcoin
sudo setfacl -Rm g:pi:rwx /mnt/bitcoin
```

We don't want to have to mount our drive every time we reboot so we'll be adding an entry to the `fstab` file so our OS knows to mount the drive on boot.

First get the UUID id:

```
sudo blkid | grep sda1
```

**NOTE** if you have other drives installed this may be sdb1 or sbc1 etc..

You should see an output that looks like:

```
/dev/sda1: UUID="XXXX-XXXX" BLOCK_SIZE="512" TYPE="exfat" PTTYPE="dos" PARTUUID="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
```

We are only interested in the line that starts with `/dev/sda1`

We can add that line to the `fstab` by opening the `fstab` file with `vim` or `nano` and adding the following line to the bottom, make sure to change the UUID to the one generated by the `blkid` command from earlier

```
vim /etc/fstab
```

```
UUID=XXXX_XXXX  /mnt/bitcoin exfat   nofail,uid=pi,gid=pi   0   0
```

After you reboot your node the drive should be mounted to `/mnt/bitcoin`

## Install Bitcoin Core

TODO: explain what bitcoin core is

### Clone the repository:

```
git clone https://github.com/bitcoin/bitcoin.git 
```

TODO: checkout stable version

### Install Berkeley DB

For backwards compatibility we will need to install and older version of Berkeley DB. Fortunately there is a script provided in the contrib folder, which we can use to install this version of BDB.

```
./contrib/install_db4.sh
```

### Install Dependencies

All of the dependencies can be found in the `build-unix.md` file located in the `docs` folder, for more information make sure to read that.

- Build requirements:

```
sudo apt install build-essential libtool autotools-dev automake pkg-config bsdmainutils

sudo apt install libevent-dev libboost-dev libboost-test-dev
```

- Install SQLite for descriptor wallet

```
sudo apt install libsqlite3-dev
```

- Optional port mapping libraries:

```
sudo apt install libminiupnpc-dev libnatpmp-dev
```

- ZMQ dependencies, used to get events out of the daemon (useful for things like block explorers):

```
sudo apt-get install libzmq3-dev
```

- User-Space, Statically Defined Tracing (USDT) dependencies:

```
sudo apt install systemtap-sdt-dev
```

- GUI dependencies (if you installed your RpiOS with a Desktop):

**NOTE** To build without GUI pass `--without-gui`

```
sudo apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools
```

- For QR Code Support:

```
sudo apt-get install libqrencode-dev
```

### Generate Build Scripts

Now we should be able to generate the build scripts, if you missed any dependencies you should be able to catch them here:

```
./autogen.sh
```

Configure:

**NOTE** The extra arguments are needed since we are using Berkeley DB.

```
export BDB_PREFIX='/home/pi/Repos/bitcoin/db4'

./configure BDB_LIBS="-L${BDB_PREFIX}/lib -ldb_cxx-4.8" BDB_CFLAGS="-I${BDB_PREFIX}/include"
```

### Compile & Install

- Compile (This could take about an hour): 

```
make
```

- Install

```
sudo make install
```

This will place the binaries in `/usr/local/bin`

```
/usr/local/bin/bitcoind

/usr/local/bin/bitcoin-cli
```

## Start and Run your Bitcoin Core Node

TODO: talk about bitcoin core vs node

### Configuration

- Let's create a config file for bitcoind

```
touch ~/.bitcoin/bitcoin.conf
```

- Add the following to your config

```
rpcuser=pi
rpcpassword=CHANGE_THIS
maxconnections=15
datadir=/mnt/bitcoin
txindex=1
```

TODO: Explain these options

- To find more config options checkout:

```
bitcoind --help
```

### Tor (Optional)

### Running bitcoind

```
bitcoind -conf=/home/pi/.bitcoin/bitcoin.conf -daemon
```

### Stopping bitcoind

```
bitcoin-cli stop
```

This will start your bitcoin node and begin downloading the entire blockchain into the `/mnt/bitcoin` directory

## References

- [Raspberry Pi Guide](https://www.htpcguides.com/properly-mount-usb-storage-raspberry-pi/) 

- [Mastering Bitcoin]() 

TODO: check out tor setup
- [Tor setup](https://8bitcoin.medium.com/how-to-run-a-bitcoin-full-node-over-tor-on-an-ubuntu-linux-virtual-machine-bdd7e9415a70) 

