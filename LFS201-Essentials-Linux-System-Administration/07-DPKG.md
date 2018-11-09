# Chapter 07 - DPKG

- [Chapter 07 - DPKG](#chapter-07---dpkg)
    - [Package File Names](#package-file-names)
    - [Source Packages](#source-packages)
    - [DPKG Queries](#dpkg-queries)

Debian Package Manager control installation, verfication, upgrade and removal of software. Similar to RPM.

Package files have a .deb suffix and the DPKG database resides in /var/lib/dpkg dir

Like rpm, dpkg program has only a partial view of the universe: it's knows what's in the system, and whatever is given on the command line.

## Package File Names

The standard naming format for a binary package is `<name>_<version>-<revision_number>_<architecture>.deb`

as in `logrotate_3.8.7-1_amd64.deb` on *Debian* and `logrotate_3.8.7-1ubuntu1_amg64.deb` on *Ubuntu*

## Source Packages

In DPKG, a source package contains at least three files:

* An upstream tarball ending with .tag.gz that contains all the source code
* A description file, ending with .dsc that contains all the metadata/architecture/dependancies
* A second tarball that contains any patch, ending with .debian.tar.gz or .diff.gz

## DPKG Queries

```bash
dpkg -l                      # List all packages installed
dpkg -L wget    # List files installed in the wget package
dpkg -s wget        # Show info about an installed package
dpkg -I <file>.deb        # Show info about a package file
dpkg -c <file>.deb          # List files in a package file
dpkg -S <file>            # Show what packages owns <file>
dpkg -s wget                # Show the status of a package
dpkg -V package # Verify the installed package's integrity
dpkg -i <file>.deb  # Installing/updgrading <file> package
dpkg -r <package>     # Remove package except config files
dpkg -P                         # Remove all package files
```
