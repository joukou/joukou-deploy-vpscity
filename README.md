Joukou VpsCity Deployment 
=========================
[![Build Status](https://circleci.com/gh/joukou/joukou-deploy-vpscity/tree/develop.png?circle-token=563e707f0719280f04e9011eef20230249af3feb)](https://circleci.com/gh/joukou/joukou-deploy-vpscity/tree/develop) [![Apache 2.0](http://img.shields.io/badge/License-Apache%202.0-brightgreen.svg)](#license)

## Initial CoreOS Installation

1. Submit a support ticket to VpsCity to mount the [CoreOS ISO - Alpha Channel](https://coreos.com/docs/running-coreos/platforms/iso/)
1. Connect to the server(s) via VNC from the VpsCity client area
1. [Install CoreOS to Disk](https://coreos.com/docs/running-coreos/bare-metal/installing-to-disk/) using the Cloud Configs in this repository
  1. If there is a live HTTPS server for https://joukou.com somewhere, that is a good place to put the configs to make them easily and securely accessible
  1. Download the Cloud Config for this server; `curl -LO https://joukou.com/cloud-config...yml`
  1. Install CoreOS to disk; `coreos-install -d /dev/vda -V current -C alpha -c cloud-config.yml -t /tmp`
  1. If you get an error "BLKRRPART device or resource busy" see Emptying partition table
1. Install [Rudder](https://github.com/coreos/rudder)
  1. Mount the disk; `mount -o subvol=root /dev/vda9 /mnt/`
  1. `mkdir -p /mnt/opt/bin`
  1. `cd /mnt/opt/bin`
  1. `curl -LO https://joukou.com/rudder`
  1. `chmod a+x rudder`
1. Change the boot order to "Hard Disk Only" from the VpsCity control panel
1. Reboot the server from the VpsCity control panel

## Installing a new version of Rudder

1. SSH into the server; `ssh core@akl..`
1. Get the Rudder sources; `git clone git@github.com:coreos/rudder.git`
1. Build Rudder; `docker run -v /home/core/rudder:/opt/rudder -i -t google/golang /bin/bash -c "cd /opt/rudder && ./build"`
1. Install the `rudder` binary; `sudo cp /home/core/rudder/bin/rudder /opt/bin/rudder`
1. Reboot; `sudo shutdown -r now` **CAUTION:** Do NOT restart multiple machines in the cluster at the same time!


## Emptying partition table

**CAUTION:** This will delete all data on disk.

1. `dd if=/dev/zero of=/dev/vda bs=1M count=100 seek=0`
1. `shutdown -r now`

## Updating Existing CoreOS Hosts

1. Login via SSH
1. Clone or pull updates from this repository
1. `sudo cp cloud-config.yml /var/lib/coreos-install/user_data`
1. `sudo shutdown -r now` NOTE: Do NOT restart multiple machines in the cluster at the same time!

## Firewall Configuration

VpsCity provides a host-level managed firewall that has been configured as
follows:

| Source IP | Protocol | Port(s) | Destination IP | Action | Purpose |
| ------------- | ----------- | ------ | ----------------------|-----------| ----------- |
| ANY | TCP | 22,80,443 | Machine's IP | ACCEPT | Public access for SSH, HTTP and HTTPS |
| Every Joukou Machine | TCP | 4001,7001 | Machine's IP | ACCEPT | Intra-cluster etcd |
| Every Joukou Machine | UDP | 8285 | Machine's IP | ACCEPT | Intra-cluster rudder |

## License

Copyright &copy; 2014 Joukou Ltd.

Joukou VpsCity Deployment is under the Apache 2.0 license. See the
[LICENSE](LICENSE) file for details.
