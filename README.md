
# Code\_Saturne / LUMA coupling on ARCHER2

This repository contains material related to running Code\_Saturne
coupled to LUMA on ARCHER2:

- [Build scripts](bin/build-all) - Scripts to install Code\_Saturne
  and LUMA on ARCHER2 from the coupling branches.
- [Tutorial](tutorial.md) - An end-user tutorial for running a coupled Code\_Saturne
  / LUMA simulation to simulate a lid-driven cavity flow with LUMA and
  Code_Saturne evolving the left and right halves of the domain
  respectively.
- [Example case](cases/ldc_left_right) - The case definition files for
  the lid-driven cavity case above
- [Components](components) - Git submodules corresponding to the
  versions of Code\_Saturne and LUMA to install.

## Building

To build the coupled codes on ARCHER2, first clone this repository on ARCHER2, either using ssh
```
git clone --recursive ssh://git@github.com/cfdemons/cs-luma-archer.git
```
or https
```
git clone --recursive https://github.com/cfdemons/cs-luma-archer.git
```

The `--recursive` flag is needed to ensure that the Code\_Saturne and
LUMA submodules are also cloned.

To install the software, choose a destination directory DESTDIR and run
```
bin/build-all DESTDIR
```

The software should be installed in the location you specified.  This
location can then be used when following the [Tutorial](tutorial.md),
rather than the location specified there.

By default, build-all installs in "development" mode, which means that
no version numbers are appended to the destination directories.  To
see the paths that would be used in release mode, use
```
bin/build-all --mode release --dry-run DESTDIR
```
and if they are OK, perform the installation with
```
bin/build-all --mode release DESTDIR
```

The codes can then be used, for example according to the
[Tutorial](tutorial.md), by setting up the environment with

```
module purge
source DESTDIR/setup.sh
```
