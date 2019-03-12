# yum

- [yum](#yum)
    - [Package Installers](#package-installers)
    - [What is yum?](#what-is-yum)
    - [Configuring yum to Use Repositories](#configuring-yum-to-use-repositories)
    - [Repository Files](#repository-files)
    - [Queries](#queries)
    - [Verifying Packages](#verifying-packages)
    - [(Un)installing/Upgrading Packages](#uninstallingupgrading-packages)
    - [Additional Commands](#additional-commands)
    - [dnf](#dnf)

yum provides a higher level of intelligent services for using the underlying **rpm** program.
It will automatically resolve dependencies when (un)installing & updating.

## Package Installers

The lower-level packages seen in previous chapters (rpm, dpkg) deals with the intracate details of installing software.

The higher-level packages management systems (yum, dnf, apt and zypper) work with database of available software. They are for

* Can use local & remote repositories as a source to install and update binary and source packages
* Are used to automate the (un)installing & update packages
* Resolve dependencies automatically

This chapter is about yum and dnf

## What is yum?

yum is basicly a front-end to rpm. Its primary task is to fetch packages from multiple remote repositories and resolve dependencies. It is used by RHEL, CentOS, Scientific Linux & Fedora.

yum caches information & db to speed up performance. yum has a number of modular extensions that can be found under `/usr/(s)bin/yum*`

## Configuring yum to Use Repositories

Repos config files are kept in `/etc/yum.repos.d` and have a .repo extension  

## Repository Files

A very simple repository file might look like

```
[repo-name]

name=Description of the repository
baseurl=http://somesystem.com/path/to/repo
enabled=1
gpgcheck=1
```

## Queries

```bash
yum search keyword      # Search for packages 
yum list                # List all packages, or just those installed, available or updates
yum info package        # Information includes size,version, where it come from, source URL and a longer description
yum grouplist           # Show information about package groups installed or available
yum groupinfo           # Show information about package groups installed or available
yum provides            # Show packages that contain a certain file name
```

## Verifying Packages

Package verification requries installation of the *yum-pluging-verify* package.

```bash
yum verify package          # verify a package, giving the most information
yum verify-rpm package      # mimic rpm -V exactly
yum verify-all package      # list all differences, including config files
```

## (Un)installing/Upgrading Packages

```bash
yum install package1            # Install one or + packages from repo
yum localinstall package-file   # Install from a local rpm
yum groupinstall group-name     # Install a specific software group
yum install @group-name         # Install a specific sofware group
yum remove package1             # Remove packages from the system
yum udpate package              # Update a package from a repository
```

## Additional Commands

```bash
yum list "yum-plugin*"                  # List all installed plugins
yum repolist                            # Show a list of all enabled repositories
yum shell text-file                     # Initiate an interactive shell in which to run multiple yum commands
yum install --downloadonly package      # Download packages but not install them
yum history                             # View the history of yum commands
```

## dnf 

dnf is intended to be next-gen replacement for yum. However, not in any major enterprise distro. 
You can test it on Fedora because it points the available commands yum -> dnf 