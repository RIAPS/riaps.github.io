
## Distribution

RIAPS runs on Linux amd64 and armhf architectures: the former is for the development environment and the control app, the latter is for the target nodes. These platforms have to pre-configured with a number of features and packages before RIAPS can be installed on them.

### Images

Note that setting up the environment for a RIAPS system is a non-trivial task - it is doable based on instructions and scripts in riaps-integration - but it takes some time. For this reason we provide two pre-built images:
- A development VM image: an amd64 Xubuntu 18.04 installation that has all packages needed for RIAPS development and control node
- An SD card image for a Beaglebone Black: an armhf Ubuntu 18.04 installation that has all packages needed on the target nodes.
These packages are available from our file server: [https://riaps.isis.vanderbilt.edu/downloads]( https://riaps.isis.vanderbilt.edu/downloads)

### Packages

RIAPS releases are distributed as deb packages from our apt repository at https://riaps.isis.vanderbilt.edu/aptrepo/. Note that there are two versions of each package: one for the development VM (-amd64) and one for the target node (-armhf). Note also that the development VM can be a target node as well. 

In the list below: ARCH = {amd64,armhf}

Package                 | Content                              | Comment
------------------------|--------------------------------------|---------
riaps-core-ARCH         | Binaries for the C++ component framework and dht-based discovery service. | 
riaps-pycom-ARCH        | Python libraries for all apps and component framework. | 
riaps-externals-ARCH    | 3rd part support libraries for RIAPS  | Typically installed once, rarely updated. 
riaps-timesync-ARCH     | Scripts and tools for time-sync service | For amd64 a physical machine with PTP-enabled NIC is required. 
riaps-systemd-ARCH      | Systemd setup scripts to launch riaps_deplo <br> as a service | 

To add this repository to an existing setup with the appropriate architecture indication (amd64 or armhf): 

```
sudo add-apt-repository -n "deb [arch=amd64] https://riaps.isis.vanderbilt.edu/aptrepo/ bionic main"
wget -qO - https://riaps.isis.vanderbilt.edu/keys/riapspublic.key | sudo apt-key add -
```
