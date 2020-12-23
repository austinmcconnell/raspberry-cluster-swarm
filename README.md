# Raspberry Pi Cluster Swarm

This installation is for building and configuring a Raspberry Pi cluster and running Docker Swarm.

I built my cluster using the [Pico 3](https://www.picocluster.com/collections/pico-3/products/pico-3-raspberry-pi).

This installation is build on top of [Hypriot OS](https://hypriot.com/) which has Docker pre-installed. Where applicable, it's built with the assumption that Macs will be used to connect/interface with the raspberry pi (e.g. choice of network mount)

## Flash OS to SD Card

### Install Flash tool
Follow the installation instructions [here](https://github.com/hypriot/flash#installation) to install the Flash tool.

### Flash the master node

Edit the user-data-master.yml file and change the password for the pirate user.

Optionally, add an authorized ssh key by adding this line to the user block.

ssh_authorized_keys:
  - ssh-rsa AAA...ZZZ (replace with your public key. If you don't know what a public key is, skip this part)

If you do provide ssh-authorized-keys and want to disable password login, change `lock_passwd` from `false` to `true` in the user-data files.

**Note: SD card should be formatted as exfat**

Format your sd card as exfat with a Master Boot Record with the following command:

```bash
diskutil eraseDisk exfat name MBRFormat /dev/diskN
```

Flash HypriotOS to your SD card by running the following command:

```bash
flash --userdata /path/to/user-data.yml https://github.com/hypriot/image-builder-rpi/releases/download/v1.9.0/hypriotos-rpi-v1.9.0.img.zip
```

If you get the error `dd: invalid number: ‘1m’` while flashing your sd card, check which version of `dd` you are using with `which dd`. If the result is, `/usr/local/opt/coreutils/libexec/gnubin/dd`, then you likely have installed `coreutils` with brew which has it's own version of dd. Uninstall `coreutils` and try the flash command again.

Insert the SD card into the raspberry pi, plug in ethernet and hdmi cables, and power up the raspbery pi. It'll take at least 5 minutes to updated all the packages.

## Flash the worker nodes

Repeat the steps above with each of the worker files (`user-data-worker1.yml`, `user-data-worker2.yml`, `user-data-worker3.yml`).


## Configure Docker Swarm
SSH into the master to get the join token

```bash
ssh pirate@pc0.local
docker swarm join-token worker
```

Copy the output of the previous command. SSH into the first worker and run the following command to join the swarm.

```bash
ssh pirate@pc1.local
docker swarm join --token $token_from_previous_step$ 192.168.1.240:2377
```

Repeat the above for workers 2 and 3
