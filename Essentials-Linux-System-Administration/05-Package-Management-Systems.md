# Package Management Systems

- [Package Management Systems](#package-management-systems)
    - [Software Packaging Concepts](#software-packaging-concepts)
    - [Why Use Packages](#why-use-packages)
    - [Package Types](#package-types)
    - [Available Package Maangement Systems](#available-package-maangement-systems)
    - [Packaging Tool Levels and Varieties](#packaging-tool-levels-and-varieties)
        - [Low level Utility](#low-level-utility)
        - [High level Utility](#high-level-utility)
    - [Package Sources](#package-sources)
    - [Creating Software Packages](#creating-software-packages)
    - [Revision Control System](#revision-control-system)

## Software Packaging Concepts

Package management systems supplys tool for: installing, upgrading, configuring and removing software packages. In details they:

* Gather and compress software into an archive
* Allow for easy installation/removal
* Verify file integrity
* Authenticate the origin of packages
* Group packages by features
* Manage dependencies between packages

## Why Use Packages

* Automation: no need for manual installation
* Scalability: install packages on multiple systemes
* Repeatability and predictability
* Security and auditing

## Package Types

They come ine several different types:

* Binary: contains files ready for deployment -> architecture-dependent and must be compiled

* Source: generate binary packages

* Architecture-independent: contains files and scripts that run under scripts interpreters

* Meta-packages: group of packages like GNOME(gui)

Binary packages are the most popular ones

## Available Package Maangement Systems

1. RPM (Red Hat Package Manager)

2. dpkg (Debian Package)

3. pacman (Arch Package Manager)

Ancient systems uses **tarballs** without any real management or clean removal strategies.

## Packaging Tool Levels and Varieties

### Low level Utility

This simply install/remove a single package (or a list)
Dependencies are not fully handled, if one needs to be installed, the installation will fail, same thing for removal.

### High level Utility

This solve the depency problems, install all the dependancies, and remove with a admin choice. (apt-get apt-cache)

## Package Sources

Every disto has their sources. There are some external repo like EPEL (Extra Package for Enterprise Linux).

## Creating Software Packages

Create a package so installing it runs scripts that perform all task needed to install the new software such as :

* Creating symbolic links
* Creating dir as needed
* Settings permissions
* Anything that can be scripted

## Revision Control System

Bigger the project, harder to control -> Source Control Systems (git, CVS, RCS, GNU Arch, Mercurial, etc.)
