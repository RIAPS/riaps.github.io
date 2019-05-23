# API Documentation Process

This document indicates how the API documentation is created.  There are two different types of documentation used: Sphinx and Doxygen.  Riaps-pycom uses Sphinx, while riaps-core and riaps-timesync use Doxygen.

## RIAPS Pycom Sphinx Documentation Process

### Environment Setup

1. Setup Sphinx tools:
```
sudo pip3 install Sphinx
sudo pip install sphinx_rtd_theme
```

2. Clone riaps-pycom repository:

  This is a working copy to create the documentation.  The resulting content in the build and source folders is never intended to be pushed back to the repository.
```
git clone https://github.com/RIAPS/riaps-pycom.git
```

### Documentation Creation

1. Build Documentation

  From within the riaps-pycom folder, run the following commands.
```
sphinx-apidoc src/riaps -o doc/source
sphinx-build -b html doc/source doc/build
```

2. Transfer HTML documentation to github.io site

  Copy the contents of the riaps-pycom/doc/build folder to the riaps.github.io/apidoc/pycom-apidoc.


## RIAPS Core Doxygen Documentation Process

### Environment Setup

1. Download Doxygen tool

```
sudo apt-get install doxygen
```

2. Clone riaps-core repository:

  This is a working copy to create the documentation.  The resulting content in the build and source folders is never intended to be pushed back to the repository.

```
https://github.com/RIAPS/riaps-core.git
```

### Documentation Creation

1. Build documentation

  From within the riaps-core/ folder, run the following command:
```
doxygen Doxyfile
```

2. Transfer HTML documentation to github.io site

  Copy the contents of the riaps-core/doc/html folder to the riaps.github.io/apidoc/core-apidoc.


## RIAPS Timesync Doxygen Documentation Process

### Environment Setup

1. Download Doxygen tool

```
sudo apt-get install doxygen
```

2. Clone riaps-timesync repository:

  This is a working copy to create the documentation.  The resulting content in the build and source folders is never intended to be pushed back to the repository.

```
https://github.com/RIAPS/riaps-timesync.git
```

### Documentation Creation

1. Build documentation

  From within the riaps-timesync/ folder, run the following commands:
```
mkdir build
cd build
cmake ..
make doc
```

2. Transfer HTML documentation to github.io site

  Copy the contents of the riaps-timesync/build/doc/html folder to the riaps.github.io/apidoc/timesync-apidoc.
