# APT

- [APT](#apt)
    - [What is APT ?](#what-is-apt)
    - [apt-get](#apt-get)
    - [Queries using apt-get](#queries-using-apt-get)
    - [(Un)installing/Upgrading packages](#uninstallingupgrading-packages)

Used on Debian-based systems, the APT (Advanced Packaging Tool) set of programs provides a higher level of intelligent services for using the udnerlying dpkg program, it plays the same role yum on Red Hat. The main utilities are apt-get and apt-cache. 

## What is APT ?

APT is not a program in itself. it includes a bunch of utilities such as **apt-get** and **apt-cache**. Those invoke the lower-level dpkg program. 

## apt-get

It's the main CLI tool for package management. It can be used to install/manage/upgrade individual packages or the entire system. It can even ugprade the distro to a completely new release. Works with multiple remote repositories. 

## Queries using apt-get

```bash
apt-cache search apache2        # Search the repository for a package named apache2
apt-cache show apache2          # Display basic information about the apache2 package
apt-cache showpkg apache2       # Display more detailed information about the apache2 package
apt-cache depends apache2       # List all dependent packages for apache2
apt-file search apache2.conf    # Search the repository for a file named apache2.conf
apt-file list apache2           # List all files in the apache2 package
```

## (Un)installing/Upgrading packages

Note that you must update before you upgrade, unlike with yum.

```bash
apt-get update              # Synchronize the package index files with their repository sources.
apt-get install             # Install new packages or update an already installed package
apt-get remove              # Remove a package from the system without removing its config files
apt-get --purge remove      # Remove a package from the system, as well as its config files
apt-get update              # Apply all available updates to packages already installed
apt-get dist-upgrade        # Do a smart upgrade taht will do a more thorough dependency resolution and remove some obsolete packages
apt-get autoremove          # Get rid of any packages not needed anymore, such as older Linux kernel versions
apt-get clean               # Clean out cache files and any archived package files that have been installed
```