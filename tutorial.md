# Code_Saturne LUMA Coupling Tutorial

This tutorial will guide you through setting up a 2D lid-driven cavity simulation run using Code_Saturne coupled to LUMA in a unit cube domain.  LUMA will evolve the portion x <= 0.55, and Code_Saturne will evolve the portion x <= 0.45.  The boundary x=0 is driven with a velocity $v_y = 1$.  Boundary data at the coupling boundaries is obtained from the other code using the PLE library.

Code_Saturne, with modifications to support coupling with LUMA, is installed in 

````bash
export CODE_SATURNE_DIR=/work/ecseaa28/ecseaa28/hinderec/projects/cs-luma/software/Code_Saturne/7.0.2/code_saturne-7.0.2/arch/Linux_x86_64
````

LUMA, also with the coupling modifications, is installed in

 ````bash
 luma_dir=/work/ecseaa28/ecseaa28/hinderec/projects/cs-luma/components/LUMA/LUMA
 ````

The lid-driven cavity coupled case definition is available at 

```bash
case_dir=/work/ecseaa28/ecseaa28/hinderec/projects/cs-luma/cases/tutorial/ldc_left_right
```

Note that while the installed version of Code_Saturne can be used directly, LUMA must be compiled for each case you want to run, as the case parameters are configured through its definitions.h header file.

First, set up the environment for running both codes:

```bash
module purge
module load PrgEnv-gnu
module load cray-python/3.8.5.0
```

The case definition looks like this:

```
ldc_left_right
├── LEFT
│   └── definitions.h
├── RIGHT
│   ├── DATA
│   │   └── setup.xml
│   ├── SRC
│   │   └── cs_user_coupling.c
│   └── cs_user_physical_properties.f90
└── run.cfg

```

The LEFT and RIGHT directories correspond to the names of the two domains (evolved by LUMA and Code_Saturne, respectively).  

definitions.h is the LUMA case definition file.

setup.xml is the standard Code_Saturne case definition file.

cs_user_coupling.c is a C source file containing the Code_Saturne configuration related to coupling.

run.cfg defines the names of the two domains to run, and the solvers (LUMA and Code_Saturne) to launch for each one.

On Archer, the case must be set up in the work directory, as the home directory is not available on the compute nodes.

Change into the directory in which you want to set up the case, and copy the case definition files into this directory.

```
cp -a $case_dir .
```

At this point, you could customise the case by editing the configuration files, but for this tutorial, we will run the case as-is.

Next, compile LUMA using the definitions.h file from the case definition:

```
cd ldc_left_right/LEFT
rsync -a --exclude input --exclude testsuite --exclude docs --exclude tools --exclude scripts $luma_dir/ build
cp definitions.h build/inc
cd build
make -B -j 8 CS_INSTALL=$CODE_SATURNE_DIR CC=CC CFLAGS="-O3 -std=c++11 -w -fopenmp -g" LAPACK_LIBS=-lsci_gnu EXE=../LUMA HDF5_HOME=/opt/cray/pe/hdf5-parallel/1.12.0.3/crayclang/9.1
cd ../..
```

==**TODO: **==

- ==In the "installed" version of LUMA, omit definitions.h entirely, so it can be overridden by an extra -I flag, maybe in CFLAGS, or with a custom variable.==
- ==Also provide a make config file which specifies all the above for Archer and the CS coupling, so the user doesn't need to provide it.==

Write a batch script to run the case:

```
#!/bin/bash

#SBATCH --job-name       ldc_left_right
#SBATCH --output         log.txt
#SBATCH --ntasks         256
#SBATCH --nodes          2
#SBATCH --cpus-per-task  1
#SBATCH --account        ecseaa28
#SBATCH --partition      standard
#SBATCH --time           01:00:00
#SBATCH --qos            lowpriority

set -eu

export LD_LIBRARY_PATH=${CODE_SATURNE_DIR}/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}
${CODE_SATURNE_DIR}/bin/code_saturne run
```

and save it in the case directory as `submit.sh`.  The simulation will run with 128 LUMA processes on one node, and 128 Code_Saturne processes on another node.  

Submit the batch job:

```
sbatch --export ALL submit.sh
```

When the job runs, log.txt will contain some logging information, but the detailed logs from the two codes will be in

```
RESU_COUPLING/*/LEFT/output_*/log_rank0.log
```

and

```
RESU_COUPLING/*/RIGHT/run_solver.log
```

The simulation should take about 25 minutes to complete.

The 3D LUMA field data will be in

```
RESU_COUPLING/*/LEFT/output_*/hdf_R0N0.h5
```

In order to visualise this in ParaView, it can be converted to VTK format:

```
cd RESU_COUPLING/*/LEFT/output_*
$luma_dir/tools/postprocessing/h5mgm/h5mgm
cd ../../../..
```

The VTK files will be generated in `RESU_COUPLING/*/LEFT/output_*/postprocessedoutput`.

The output of the simulation can be visualised in ParaView.  You can either run ParaView in client-server mode, with pvserver running on Archer, and a local ParaView client running on your own computer, or you can copy the case output data to your own computer and visualise them directly with ParaView running there.  Documentation for using ParaView in client-server mode is available [here](https://docs.archer2.ac.uk/data-tools/paraview/).

Launch ParaView and open the LUMA Fluid timeseries from `RESU_COUPLING/*/LEFT/output_*/postprocessedoutput`.

Next, open the Code_Saturne fluid data from `RESU_COUPLING/*/RIGHT/postprocessing/RESULTS_FLUID_DOMAIN.case`.

Make a plot of the  $y$ component of velocity from each dataset.

The LUMA data will need to be rescaled by a factor of 0.005/0.001 to convert from Lattice Boltzmann units to physical units.

The LUMA data will need timestep information added, to ensure it is in sync with the Code_Saturne data.

You should obtain the following plot.

