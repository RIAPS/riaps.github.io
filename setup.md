
# RIAPS Setup

RIAPS runs on x86_64/amd64 and armhf platforms - the first is for the development, the second for deployment. 
On both platforms we use Ubuntu 18.04 as the foundation, and RIAPS is layered on top of that.

RIAPS is available in various forms.
* Pre-built development VM and BBB SD card images
* Debian packages via an apt repository
* Source code in github repositories

The easiest way to get started is to use the pre-built images - follow the instructions on [RIAPS Integration](https://github.com/RIAPS/riaps-integration/blob/master/README.md) page. These images take some time to build due to the number of packages RIAPS uses, but that repository has the instructions and supporting scripts. 

The images have a recent version of RIAPS pre-installed, but as the system is being developed those need to be updated. This happens via our apt repository where we periodically produce new releases as Debian (.deb) packages. This repository is already added to the apt source list on the images, i.e. the usual 'apt-get' update mechanism will update them. The deb packages can also be downloaded from the repository release pages and installed via a 'dpkg -i PACKAGE.deb' command. 

The github repositories for the project contain the source code for all the packages, and the README files give instructions on how to build them.
