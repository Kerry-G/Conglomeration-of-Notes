# RPM

- [RPM](#rpm)
    - [Advantages of RPM](#advantages-of-rpm)
        - [Sys admins](#sys-admins)
        - [Dev.](#dev)
    - [Package File Name](#package-file-name)
    - [Database directory](#database-directory)
    - [Helper Programs and modifying settings](#helper-programs-and-modifying-settings)
    - [Queries](#queries)
    - [Verifying Packages](#verifying-packages)
    - [Installing Packages](#installing-packages)
    - [Uninstalling Packages](#uninstalling-packages)
    - [Upgrading Packages](#upgrading-packages)
    - [Freshening Packages](#freshening-packages)
    - [Upgrading the Kernel](#upgrading-the-kernel)
    - [Using rpm2cpio](#using-rpm2cpio)

Red Hat Package Manager control installation, verfication, upgrade and removal of software.

All files related to a specific task are grouped into a rpm file. New version of software equals to a new rpm file.
They also contain dependency information. They need specifics URL to draw the software from. Usually distro-dependent.

## Advantages of RPM

### Sys admins

* Determine what package any file on the system is part of
* Determine what version is installed
* (Un)Install packages
* Verify packages
* Distinguish doc files and opt-out of them for spaces
* use FTP or HTTP to install packages

### Dev.

* Software often available for more than one OS. RPM is used for the basis but a dev can separate out the changes needed.
* More than one architecture can be built using only one source package.

## Package File Name

The standard naming format for a bin package is <name>-<version>-<release>-.<distro>.<architecture>.rpm
ex: sed-4.2.1-10.el6.x86_64.rpm

The standard naming format for a source package is <name>-<version>-<release>-.<distro>.src.rpm
ex: sed-4.2.1-10.el6.src.rpm

## Database directory

/var/lib/rpm is the default system directory which hold RPM database files in the form of Berkeley DB hash files.
The DB should not be directly modified; only through RPM program.

An alt. database can be probided with the --dbpath option, and a rebuild of the db can be done by using --rebuilddb

## Helper Programs and modifying settings

Helper programs used by rpm reside in /usr/lib/rpm. A rpmrc file can be created to specify rpm's settings.
By default rpm looks for /usr/lib/rpm/rpmrc, /etc/rpmrc and ~/.rpmrc

It will not stop as soon as it finds one. Can also provide one manually with the --rcfile option

## Queries

All queries starts with -q

* Version of packages is installed: `rpm -q bash`
* Which package did this file comes from: `rpm -qf /bin/bash`
* What files were installed by this package: `rpm -ql bash`
* Show information about this package: `rpm -qi bash`
* Show information about this pacakge from the package file, not the package database: `rpm -qip foo-1.0.0-1.noarch.rpm`
* List all installed packages: `rpm -qa`
* Return a list of prereq. for a package :`rpm -qb --requires foo-1.0.0-1.noarch.rpm`
* Show what installed package provide sa particular requisite package: `rpm -q --whatprovides libc.so.6`

## Verifying Packages

the -V option allows us to verify whether the files from a package are consistent with the system's RPM database.

Verify all packages installed : `rpm -Va`

It will give us a list with a couple characters, the meaning of them are:

* S: file size differs
* M: file permissions/type differs
* 5: MD5 checksum differs
* D: Device major/minor number mismatch
* L: Symboloi links path mismatch
* U: User ownership differs
* G: Group ownership differs
* T: modification time differs
* P: capabilities differ

You can also specify packages names to -V: `rpm -V bash`

## Installing Packages

As simple as: `sudo rpm -ivh foo-1.0.0-1.noarch.rpm`

* -i: install
* -v: verbose
* -h: print out hash marks while doing to show progress

RPM will:

* Performs dependency checks
* Performs conflict checks
* Executes commands required before installation
* Deals intelligently with config files
* Unpacks the files from packages and installs them with correct attributes
* Executes commands required post-installation
* Updates the system RPM database

## Uninstalling Packages

Uninstall a packages with the -e options: `sudo rpm -e system-config-lvm`

You can test if the uninstallation will fail or not by giving the --test option along with -e. NEVER REMOVE THE RPM package.

## Upgrading Packages

You can upgrade a package with the -U option: `sudo rpm -Uvh bash-4.2.45-5.el7_0.4.x86_64.rpm`

You can downgrade the package with the --oldpackage option.

## Freshening Packages

`sudo rpm -Fvh *.rpm` will attempt to freshen all the packages in the current dir.

1) If an older version of a package is installed, it will be upgraded
2) If the version in the system is the same, nothing happens
3) If there is no version of a package installed, package in dir is ignored.

## Upgrading the Kernel

To upgrade a kernel, one does not simply -U the kernel, but instead install the newer kernell, boot into it and then delete the older one.

It is one of the rare upgrade that requires a reboot.

## Using rpm2cpio

Let's say you want to extract files from a rpm but you don't want to install it

* Create the cpio archive with: `rpm2cpio foobar.rpm > foobar.cpio`
* To list files in a rpm: `rpm2cpio foobar.rpm | cpio-t`
* A better way to do this: `rpm -qilp foobar.rpm`
* To extract onto the system: `rpm2cpio foorbar.rpm | cpio --extract --make-directories`
