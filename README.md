# Operations Management Client (Build-SCXcore)

##### Table of Contents
- [Glossary of Terms](#glossary-of-terms)
- [Setting up a machine to build SCX](#setting-up-a-machine-to-build-scx)
  - [Sudoers configuration](#sudoers-configuration)
  - [Dependencies to build a native package](#dependencies-to-build-a-native-package)
  - [Setting up a system to build a shell bundle](#setting-up-a-system-to-build-a-shell-bundle)
- [Cloning the repository](#cloning-the-repository)
- [Building the agent](#building-the-agent)
- [Code of Conduct](#code-of-conduct)

If you are an active contributor to the SCXcore project, you should
[set up your system](https://github.com/Microsoft/ostc-docs/blob/master/setup-git.md)
and follow our [common workflow](https://github.com/Microsoft/ostc-docs/blob/master/workflow-workflow.md).
New to git? Read [guidelines for development](https://github.com/Microsoft/ostc-docs/blob/master/setup-rules.md).

-----

### Glossary of Terms 

A short glossary might be helpful if this project is new to you:

Term | Meaning
---- | -------
SCX | This is a historical term (System Center X-Plat (Cross Platform)), when SCX was the only product of its type. Now there are many different providers for many different purposes, and SCX is just one provider.
Shell Bundle | A "shell bundle" is a distribution mechanism for the SCX agent. A shell bundle is a shell script that has, as part of it, .RPM and .DEB native package files for OMI and SCX itself. The shell bundle determines what has to be installed based on system configuration and installs the appropriate packages.<br><br>Note that the SCX shell bundle contains other shell bundles to install other packages as well (Apache and MySQL, for example). Due to this, we consider the SCX shell bundle to be a "mega-bundle".
ULinux | A "Universal Linux" build is a type of build that will install and run on any Linux system that we support. We have two universal builds for Linux: One for 32-bit systems, and one for 64-bit systems.

-----

### Setting up a machine to build SCX

There are two ways to build SCX:

1. As an RPM or DEB package that can be installed on the local system,
assuming all dependencies are met (most easily met if a shell bundle
was previously installed to "seed" the system), or

2. As a shell bundle.

Building a shell bundle is a superset of setting up a system to build a local RPM, so we will cover that first.

#### Sudoers configuration

[Configure sudo](https://github.com/Microsoft/ostc-docs/blob/master/setup-build.md)

#### Dependencies to build a native package

Note that it's very nice to be able to use the [updatedns](https://github.com/jeffaco/msft-updatedns) project to use host names
rather than IP numbers in a Hyper-V environment. On CentOS systems,
this requires the bind-utils package (updatedns requires the `dig`
program). The bind-utils package isn't otherwise necessary.

- On CentOS 7.x
```
 sudo yum install git bind-utils gcc-c++ rpm-devel pam-devel openssl-devel rpm-build
```
- On Ubuntu 14.04
```
 sudo apt-get install git pkg-config make g++ rpm librpm-dev libpam0g-dev libssl-dev
```

- Notes on other platforms

 When building a machine for ULINUX builds (such as SuSE 10), we
 suggest using the O/S distribution CD to install the packages. It's
 not as easy, but that's the only way to guarantee that packages
 aren't updated such that generated binaries are not backwards
 compatible. (See notes on building a shell bundle, elsewhere in this
 document.) Note that ULinux builds are controlled via the configure
 script, discussed below.

 Also note that since you won't use `yum`, you must also handle the dependent
 packages manually (keep adding lines to the `rpm install` command line until
 all dependencies are satisfied).

 Similar methods would be utilized if building a Redhat system that is not
 registered for use for up2date.

#### Setting up a system to build a shell bundle

To build a shell bundle, we need and older Linux system (we typically use
SuSE 10.0 for this), as binary images created with older Linux systems
are generally upwards compatible when installed on newer Linux systems.

A notable exception: We use the OpenSSL package, and we can't tell if
we need OpenSSL v0.9.8 or OpenSSL v1.0.x. As a result, we have a [special process](https://github.com/Microsoft/ostc-openssl/blob/master/README.md) to build both versions of OpenSSL that we can link against.

Once OpenSSL is set up, you need to configure omsagent to include the
```--enable-ulinux``` qualifier, like this:<br>```./configure --enable-ulinux``` 

### Cloning the repository

Note that there are several subprojects, and authentication is a hassle
unless you set up an SSH key via your GitHub account. [Set up your machine](https://github.com/Microsoft/ostc-docs/blob/master/setup-git.md) properly for a much easier workflow.

To clone the repository to build SCXcore, issue the following command:

```
git clone --recursive git@github.com:Microsoft/Build-SCXcore.git bld-scxcore
```

After this, you need to make sure that you're on the master branch for each
of the subprojects. To do this, issue the following commands:

```
cd bld-scxcore
git checkout master
git submodule foreach git checkout master
```

You can also use an alias like ```git co-master``` if you followed 
[Configuring git](https://github.com/Microsoft/ostc-docs/blob/master/setup-git.md)
recommendations.


### Building the Agent

From the `bld-scxcore` directory (created above from `git clone`), do the
following:

```
cd opsmgr/build
./configure
make
make test
```

Note that the ```configure``` script takes a variety of options. You
can use ```configure --help``` to see the options available.

When the build completes, you should have a native package that you
can install on your system. The native package should be in a
subdirectory off of `bld-scxcore/opsmgr/intermediate` (the directory
name varies based on debug vs. release builds).

As mentioned above, this form of build requires a shell bundle to be
installed to "seed" your system with other required dependencies.
Alternatively, you can simply install the OMI and SCX packages for
your system, in that order.

Note that a shell bundle can install more than OMI and SCX on your
system. Depending on your system configuration, providers for Apache
and MySQL may also be installed.

### Code of Conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).  For more
information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact
[opencode@microsoft.com](mailto:opencode@microsoft.com) with any
additional questions or comments.
