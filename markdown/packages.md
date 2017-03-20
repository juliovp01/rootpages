# Packages

* DEB - Debian
* [RPM - RHEL](#rpm---rhel)
    * [Adding a Repository](#rpm---adding-a-repository)
    * [Creating a Repository](#rpm---creating-a-repository)
    * [Creating an RPM](#rpm---creating-an-rpm)
        * [Directories](#rpm---creating-an-rpm---directories)
        * [Users and Groups](#rpm---creating-an-rpm---users-and-groups)
    * Repositories
        * Official (EPEL)
        * Unofficial (Repoforge)
* [PKGBUILD - Arch](#pkgbuild---arch)
    * Repositories
        * Official (ABS, Delta)
        * Unofficial (AUR)
* Flatpak


# RPM - RHEL


## RPM - Adding a Repository

Repositories (sometimes called "repos") are a central location where packages can easily be found and installed from.

On Red Hat based systems, the repositories are all defined as text files with the ".repo" extension in this directory.
```
# ls /etc/yum.repos.d/
```

Here are some common options for repository files:
* [] = This should be the first part of a repository, with the name being inside the brackets.
* name = This should be similar to the name from the brackets. However, this friendly name can be different and is usually ignored.
* baseurl = The location of the repository. Valid location types include "http://", "ftp://", and "file://" for using the local file system.
* mirrorlist = Instead of a baseurl, a link to a list of repository mirrors can be given.
* enabled = Enable or disable a repository with a "1" or "0". The default is value is "1". [1]
* gpgcheck = Force a GPG encryption check against signed packages. Enable or disable with a "1" or "0".
* gpgkey = Specify the path to the GPG key.

Variables for repository files:
* $releasever = The RHEL release version. This is typically the major operating system versioning number such as "5" or "6".
* $basearch = The CPU architecture. For most modern PCs this is typically either automatically filled in as "x86_64" for 64-bit operating systems or "i386" for 32-bit. [2]

At the bare minimum, a repository file needs to include a name and a baseurl.
```
[example-repo]
name=example-repo
baseurl=file:///var/www/html/example-repo/
```

Here is an example repository file for the official CentOS 7 repository using a mirrorlist.
```
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

Sources:

1. "Fedora 24 System Administrator's Guide" Fedora Documentation. 2016. Accessed June 28, 2016. https://docs.fedoraproject.org/en-US/Fedora/24/html/System_Administrators_Guide/sec-Setting_repository_Options.html
2. "yum.conf - Configuration file for yum(8)." Die. Accessed June 28, 2016. http://linux.die.net/man/5/yum.conf


## RPM - Creating a Repository

Any directory can be used as a repository to host RPMs. The standard naming convention used for RHEL based operating systems is "centos/$releasever/$basearch/" where $releasever is the release version and $basearch is the CPU architecture. However, any directory can be used.

In this example, a default Apache web server will have the repository access via the URL "http://localhost/centos/7/x86_64/." Be sure to place your RPMs in this directory. [1]
```
# yum install createrepo
# mkdir -p /var/www/html/centos/7/x86_64/
```
```
# createrepo /var/www/html/centos/7/x86_64/
```

The "createrepo" command will create 4 or 5 files.
* repomd.xml = An index for the other repository metadata files.
* primary.xml = Contains metadata for all packages including the name, version, architecture, file sizes, checksums, dependencies, etc.
* filelists.xml = Contains the full listing of every directory and file.
* other.xml = Holds a changelog of all the packages.
* groups.xml = If a repository has a "group" that should install multiple packages, the group is specified here. By default, this file is not created when running "createrepo" without any arguments. [2]

If new packages are added and/or signed via a GPG key then the repository cache needs to be updated again. [1]
```
# createrepo --update /var/www/html/centos/7/x86_64/
```

Sources:

1. "createrepo(8) - Linux man page." Die. Accessed June 28, 2016. http://linux.die.net/man/8/createrepo
2. "createrepo/rpm metadata." createrepo. Accessed June 28 2016. http://createrepo.baseurl.org/


## RPM - Creating an RPM

Create a SPEC file. This modified shell script contains all of the information about the program and on how to install and uninstall it. It is used to build the RPM.

Common variables:

* Name = The name of the program.
* Version = The version of the package. Typically this is in the format of X.Y.Z (major.minor.bugfix) or ISO date format (for example, "2016-01-01").
* Release = Start with "1%{?dist}" for the first build of the RPM. Increase the number if the package is ever rebuilt. Start from "1%{?dist}" if a new version of the actual program is being built.
* Summary = One sentence describing the package. A period is not allowed at the end.
* BuildRoot = The directory that contains all of the RPM packages. The directory structure under here should mirror the files location in relation to the top-level root "/". For example, "/bin/bash" would be placed under "$RPM_BUILD_ROOT/bin/bash".
* BuildArch = The architecture that the program is meant to run on. This is generally either "x86_64" or "i386". If the code is not dependent on the CPU (for example, Java or bash scripts) then "noarch" can be used.
* Requires = List the RPM packages that are dependencies needed for your program to work.
* License = The license of the program.
* URL = A URL link to the program's or, if that is not available, the developer's website.
* Source = A tarball of the source code. It should follow the naming standard of `<RPM_NAME>-<RPM_PROGRAM_VERSION>.tar.gz`.

Sample SPEC file:
```
Name: my-first-rpm
Version: 1.0.0
Release: 1%{?dist}
Summary: This is my first RPM
License: GPLv3
URL: http://example.tld/
```
If you want to build the RPM, simply run:
```
# rpmbuild -bb <SPECFILE>.spec
```
In case you also want to build a source RPM (SRPM) run:
```
# rpmbuild -ba <SPECFILE>.spec
```

Sections:

* %description = Provide a description of the program.
* %build = This is where the program is built from the source code.
* %install = Copy files to a directory structure under $RPM_BUILD_ROOT that mirrors where their installed location. The $RPM_BUILD_ROOT is the top-level directory of a typical Linux file system hiercharchy.
* %file = These are the files that should be copied over during installation. Permissions can also be set.
    * `%attr(<MODE>, <USER>, <GROUP>)` = Define this in front of a file or folder to give it custom permissinos.

Source:

1. "How to create an RPM package." Fedora Project. June 22, 2016. Accessed June 28, 2016. http://fedoraproject.org/wiki/How_to_create_an_RPM_package


### RPM - Creating an RPM - Directories

During the creation of an RPM there are a few important directories that can and will be refereneced.

* %{topdir} = The directory that the RPM related files should be located. By default this is set to `%{getenv:HOME}/rpmbuild`.
* %{builddir} = The `%{_topdir}/BUILD` directory. This is where the compilation of the program should take place.
* %{_sourcedir} = The `%{_topdir}/SOURCES` directory. This is where patches, service files, and source code can be stored.
* %{_specdir} = The `%{_topdir}/SPECS` directory. This is where the SPEC file for the RPM should be stored.
* %{_srcrpmdir} = The `%{_topdir}/SRPMS` directory. This is where the optional source RPM will be compiled and stored to.
* %{buildroot} = The `%{_topdir}/BUILDROOT` directory. This is the file system hierarchy of where the RPM files will actually be installed to. This is also set to the `$RPM_BUILD_ROOT` shell variable.

[1]

Source:

1. "Packaging:RPMMacros." Fedora Project Wiki. December 1, 2016. Accessed March 13, 2017. https://fedoraproject.org/wiki/Packaging:RPMMacros?rd=Packaging/RPMMacros


### RPM - Creating an RPM - Users and Groups

Creating a user or group can be done one of two ways.

* Dynamically = Let the system deciede what user identification number (UID) and group ID (GID) to use.
* Static = Specify a specific UID or GID number to use. This is useful for keeping permissions identifical on multiple platforms.

The Fedora Project recommends using these standarized blocks of code to accomplish these methods. [1]

Dynamic:
```
Requires(pre): shadow-utils
[...]
%pre
getent group <GROUP_NAME> >/dev/null || groupadd -r <GROUP_NAME>
getent passwd <USER_NAME> >/dev/null || \
    useradd -r -g <GROUP_NAME> -s /sbin/nologin \
    -c "<USER_DESCRIPTION>" <USER_NAME>
exit 0
```

Static:
```
Requires(pre): shadow-utils
<OMITTED>
%pre
getent group <GROUP_NAME> >/dev/null || groupadd -f -g <GID> -r <GROUP_NAME>
if ! getent passwd <USER_NAME> >/dev/null ; then
    if ! getent passwd <UID> >/dev/null ; then
      useradd -r -u <UID> -g <GROUP_NAME> -s /sbin/nologin -c "Useful comment about the purpose of this account" <USER_NAME>
    else
      useradd -r -g <GROUP_NAME> -s /sbin/nologin -c "<USER_DESCRIPTION>" <USER_NAME>
    fi
fi
exit 0
```

Source:

1. "Packaging: Users and Groups" Fedora Project. September 14, 2016. Accessed February 25, 2017. https://fedoraproject.org/wiki/Packaging:UsersAndGroups


## PKGBUILD - Arch

Arch Linux packages are design to be simple and easy to create. A PKGBUILD file is compressed with a software's contents into a XZ tarball. This can contain either the source code or compiled program.

Required Variables:

* pkgname = Name of the software.
* pkgver = Version of the software.
* pkgrel = Version of the package (only increase if the PKGBUILD file has been modified and not the software).
* arch = The architecture the software is built for. Any architecture that applies should be defined. Valid options: x86_64, i686, arm (armv5), armv6h, armv7h, aarch64 (armv8 64-bit), or any.

Optional Variables:

* pkgdesc = A brief description of the software.
* url = The URL of the software's website.
* license = The license of the software. Valid options: GPL, BSD, MIT, Apache, etc.
* depends = List other package version dependencies.
* optdepends = List optional dependencies and a brief description.
* makedepends = List packages required to build the software from source.
* provides = List tools that are provided by the package but do not necessarily have file names.
* conflicts = List any conflicting packages.
* replaces = List packages that this software should replace.

[1]

Functions

Required:

* build()
    * For building the software, PKGBUILD will need to move into the directory that the XZ tarball was extracted to. This is automatically generated as the "srcdir" variable. In most situations this should be the package name and version separated by a dash.
```
$ cd "${srcdir}"
```
OR
```
$ cd "${pkgname}-${pkgver}"
```

* package()
    * These are the steps to copy and/or modify files from the "srcdir" to be placed in the "pkgdir" to represent where they will be installed on an end-user's system. This acts as the top-level directory of a Linux filesystem hierarchy.
```
$ cd "${pkgdir}"
```
    * An example of installing compiled source code using a Make file.
```
$ make DESTDIR="${pkgdir}" install
```
[2][3]

Sources:

1. "PKGBUILD." Arch Linux Wiki. October 26, 2016. Accessed November 19, 2016. https://wiki.archlinux.org/index.php/PKGBUILD
2. "Creating packages." Arch Linux Wiki. July 30, 2016. Accessed November 19, 2016. https://wiki.archlinux.org/index.php/creating_packages
3. "PKGBUILD(5) Manual Page." Arch Linux Man Pages. February 26, 2016. Accessed November 19, 2016. https://www.archlinux.org/pacman/PKGBUILD.5.html