
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

Modify [bin/build-all](bin/build-all) to customise the installation
location, `destdir`.  Then, run
```
bin/build-all
```

The software should be installed in the location you specified.  This
location can then be used when following the [Tutorial](tutorial.md),
rather than the location specified there.
