
## Distribution

RIAPS runs on Linux amd64, armhf and arm64 architectures: the first one is for the development environment and the control app, the last two are for the target nodes. These platforms have to be pre-configured with a number of features and packages before RIAPS can be installed on them.

### Images

Note that setting up the environment for a RIAPS system is a non-trivial task - it is doable based on instructions and scripts in riaps-integration - but it takes some time. For this reason we provide pre-built images:
- A development VM image: an amd64 Xubuntu 20.04 installation that has all packages needed for RIAPS development and control node
- Target node SD card images for the following single board computers. Each has an Ubuntu 20.04 installation that has all packages needed for the RIAPS platform.  These packages are available from our file server: [https://riaps.isis.vanderbilt.edu/downloads]( https://riaps.isis.vanderbilt.edu/downloads)
  - Beaglebone Black: armhf
  - Raspberry Pi 4: arm64
  - Jetson Nano: arm64

### Packages

RIAPS releases are distributed as deb packages from our apt repository at https://riaps.isis.vanderbilt.edu/aptrepo/. Note that there are three versions of each package: one for the development VM (-amd64) and two for the target nodes (-armhf and -arm64). Note also that the development VM can be a target node as well.

In the list below: ARCH = {amd64,armhf,arm64}

Package                 | Content                              | Comment
------------------------|--------------------------------------|---------
riaps-pycom-ARCH        | Python libraries for all apps and component framework. |
riaps-timesync-ARCH     | Scripts and tools for time-sync service | For amd64 a physical machine with PTP-enabled NIC is required.

To add this repository to an existing setup with the appropriate architecture indication (amd64, armhf or arm64):

```
wget -qO - https://riaps.isis.vanderbilt.edu/keys/riapspublic.key | sudo apt-key add -
sudo add-apt-repository -n "deb [arch=amd64] https://riaps.isis.vanderbilt.edu/aptrepo/ focal main"
```
