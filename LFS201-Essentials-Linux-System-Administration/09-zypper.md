# zypper

- [zypper](#zypper)
    - [What is zypper?](#what-is-zypper)
    - [Queries](#queries)
    - [(Un)installing/Upgrading packages](#uninstallingupgrading-packages)
    - [Additionals commands](#additionals-commands)

For SUSE-based systems, zypper program provides a higher level of intelligent services for **rpm**. It is the same as yum on Red Hat.

## What is zypper?

CLI tool for managing packages in SUSE Linux. It's very similar to yum. 

## Queries

```bash
zypper list-updates                 # Show a list of available updates
zypper repos                        # List available repositories
zypper search <string>              # Search repositories for string
zypper info <package>               # List information about package
zypper search --provides <file>     # Search repositories to ascertain what packages provide a file
```

## (Un)installing/Upgrading packages

```bash
zypper install package                          # Install or update package
zypper --non-interactive install <package>      # Do not ask for confirmation when installing or upgrading
zypper update                                   # Update all installed packages
zypper --non-interactive update                 # Giving package names as an argument will update only those packages and any requried dependencies
zypper remove <package>                         # Remove a package from the system
```

## Additionals commands

```bash
zypper addrepo URI alias        # add a new repository
zypper removerepo alias         # remove a repository from the list
```