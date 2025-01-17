Running Galaxy Simulations
**********


Cosmological Simulations (Large-Box)
============

Initial Conditions With MUSIC
-----------------

The very first thing we'll need to do is to set up initial conditions
with MUSIC (https://www-n.oca.eu/ohahn/MUSIC/).  For MUSIC, you'll
need a few libraries (compiler, GSL and FFTW loaded at the least).  I
suggest having your flags and paths set in the compiler as something
like::

  ##############################################################################
  ### compiler and path settings
  CC      = icc
  OPT     = -Wall -Wno-unknown-pragmas -O3 -g -mtune=native
  CFLAGS  =
  LFLAGS  = -lgsl -lgslcblas #-ldrfftw_threads
  CPATHS  = -I. -I$(HPC_FFTW_INC) -I$(HPC_GSL_INC)
  LPATHS  = -L$(HPC_FFTW_LIB) -L$(HPC_GSL_LIB)

So that the code automagically looks for whatever is added to your path when you module load the libraries.  In principle, you can just load modules before compiling like::

  module purge
  module load intel
  module load openmpi
  module load gsl
  module load hdf5
  module load fftw

The next thing you'll need is a configuration file for MUSIC.  Let's
set up a config file for a 25/h Mpc (side-length) and 512^3 (particle number) box.  The config file could look something like this::

  [desika.narayanan@login1 ICs]$ pwd
  /orange/narayanan/desika.narayanan/gizmo_runs/simba/m25n512/ICs

  [desika.narayanan@login1 ICs]$ cat ics_m25n512.conf
  [setup]
  boxlength		= 25
  zstart			= 249
  levelmin		= 9
  levelmin_TF		= 9
  levelmax		= 9
  padding			= 8
  overlap			= 4
  ref_center              = 0,0,0
  ref_extent              = 0,0,0
  align_top		= no
  baryons			= yes
  use_2LPT		= no
  use_LLA			= no
  periodic_TF		= yes
  
  [cosmology]
  Omega_m			= 0.3
  Omega_L			= 0.7
  Omega_b			= 0.048
  H0			= 68.0
  sigma_8			= 0.82
  nspec			= 0.97
  transfer		= eisenstein
  
  [random]
  seed[9]			= 8675309
  
  [output]
  ##Gadget-2 (type=1: high-res particles, type=5: rest)
  format			= gadget2
  filename		= ics_m25n512
  gadget_usekpc		= yes
  gadget_usemsol		= no

  [poisson]
  fft_fine		= yes
  accuracy		= 1e-5
  pre_smooth		= 3
  post_smooth		= 3
  smoother		= gs
  laplace_order		= 6
  grad_order		= 6

Now note, there are a ton of options not listed here (that work both
with other hydrocodes than gadget-oids, as well as even for gadget
itself, and you should check out the MUSIC manual for those).  But in
short, the [setup] region of this tells you some obvious basics -- box
size, what redshift should the IC be set up for, what is the
coordinate system, etc.  The levelmin/max stuff is the particle count
-- so 9==2^9==512.  Similarly, we set that we want baryons (unless, of
course, we don't...) and our cosmology.  Important: this cosmology
will need to be the same as what we use in our actual hydro simulation.

Once this config file is set, we need to actually run MUSIC on the config file to create the IC::

  [desika.narayanan@login1 ICs]$ pwd
  /orange/narayanan/desika.narayanan/gizmo_runs/simba/m25n512/ICs

  [desika.narayanan@login1 ICs]$ cat music.job
  #!/bin/bash
  #SBATCH --job-name=music
  #SBATCH --output=music.o
  #SBATCH --error=music.e
  #SBATCH --mail-type=ALL
  #SBATCH --mail-user=desika.narayanan@gmail.com
  #SBATCH --time=36:00:00
  #SBATCH --ntasks=1
  #SBATCH --cpus-per-task=32
  #SBATCH -N 1
  #SBATCH --mem-per-cpu=3800
  #SBATCH --account=narayanan
  #SBATCH --qos=narayanan-b
  
  module purge
  module load intel
  module load openmpi
  module load gsl
  module load hdf5
  module load fftw
  ./MUSIC ics_m25n512.conf

and the resultant file (which we set in the .conf file to be ics_m25n512) is the HDF5 initial condition for the simulation!

Compiling Gizmo
-----------------

We next want to run the actual gizmo simulation.  You'll need to clone
the gizmo repository.  Typically we've been using the SIMBA set of
galaxy physics, which you can find here:
https://bitbucket.org/romeeld/gizmo-mufasa/src/master/ (note, this is
private so you'll need access).

To comppile, the first thing we need is a Makefile that is set for our
system.  Edit Makefile.systype to have evverythign commented out
except the system we plan on using.  For example::

  # Select Target Computer
  #
  # Please copy this file to Makefile.systype and uncomment your
  # system. Don't commit changes to this file unless you add support for
  # a new system.
  #
  ###########
  #
  # This file was originally part of the GADGET3 code developed by
  #   Volker Springel (volker.springel@h-its.org).
  #
  #############
  
  ###################
  ## RT/RD SYSTEMS ##
  ###################
  #SYSTYPE="RTOSX"
  #SYSTYPE="ELGATO-GNU"
  #SYSTYPE="ELGATO-INTEL"
  #SYSTYPE="TIMON-PUMBAA_GNU"
  #SYSTYPE="TIMON-PUMBAA_OPEN64"
  #SYSTYPE="ursa"
  #SYSTYPE="ursa-open64"
  #SYSTYPE="fock"
  #SYSTYPE="fockgnu"
  SYSTYPE="hipergator-intel"
  #SYSTYPE="hipergator-gnu"
  #SYSTYPE="archer"
  #SYSTYPE="cosma-intel"
  #SYSTYPE="cosma-gnu"
  ################
  
  #SYSTYPE="Stampede"
  #SYSTYPE="Zwicky"
  #SYSTYPE="MacBookPro"
  #SYSTYPE="Quest"
  #SYSTYPE="odyssey"
  #SYSTYPE="SciNet"
  #SYSTYPE="Pleiades-Haswell"
  #SYSTYPE="Pleiades-SIBridge"
  #SYSTYPE="Ranger_intel"
  #SYSTYPE="Ranger_pgi"
  #SYSTYPE="Darwin"
  #SYSTYPE="Magny"
  #SYSTYPE="Magny-Intel"
  #SYSTYPE="OpenSuse"
  #SYSTYPE="OpenSuse64"
  #SYSTYPE="HLRB2"
  #SYSTYPE="MPA"
  #SYSTYPE="VIP"
  #SYSTYPE="Ubuntu"
  #SYSTYPE="MBM"
  #SYSTYPE="OpteronMPA-Gnu"
  #SYSTYPE="OpteronMPA-Intel"
  #SYSTYPE="Centos5-intel"
  #SYSTYPE="Kolob"
  #SYSTYPE="Centos5-Gnu"
  #SYSTYPE="OPA-Cluster64-Intel"


Where, here, we are obviously saying we'll compile using intel
compilers on HPG.  The next thing to do is to ensure that there are
actually system directives in the Makefile to actually compile!   For example, in the Makefile, have something like::

  ifeq ($(SYSTYPE),"hipergator-intel")
  CC   =  mpicc
  CXX  =  mpicxx
  FC   =  $(CC)
  OPT += -DH5_USE_16_API #-DCONFIG_BFLOAT_8
  #GSL_INCL    = -I$(HPC_GSL_INC)
  GSL_INCL    = -I/apps/intel/2018.1.163/gsl/2.4/include
  GSL_LIBS    = -L$(HPC_GSL_LIB)
  FFTW_HOME   = /apps/intel/2018.1.163/openmpi/3.1.0/fftw/2.1.5/
  FFTW_INCL   = -I$(FFTW_HOME)/include
  FFTW_LIBS   = -L$(FFTW_HOME)/lib64
  HDF5LIB     = -L$(HPC_HDF5_LIB) -lhdf5
  HDF5INCL    = -I$(HPC_HDF5_INC)
  BLAS_LIBS   = -L$(HPC_MKL_LIB) -lmkl_intel_lp64 -lmkl_sequential -lmkl_core
  GRACKLEINCL = -I$(HPC_GRACKLE_INC)
  GRACKLELIBS = -L$(HPC_GRACKLE_LIB) -lgrackle

Finally, we'll need to make some decisions about how to actually run
gizmo, given the physics that is implemented in the fork that we have.
This is really going to depend on your specific needs, so there's no
catch-all solution here. You can get the default Config.sh from the
simba-gizmo site.


Now, we should be able to compile!  Load the following modules, and
compile!::

  module purge
  module load intel/2018
  module load hdf5/1.10.1
  module load openmpi/3.1.2
  module load gsl/2.4
  module load fftw/2.1.5
  module load grackle

Once it's compiled, there is a parameter file to edit.  This will
point to your IC file, your output directory.  Some other things
you'll need to think about are the softening lengths: a reasonable
default is box length/particles per side/200 (in Mpc).  There's a nice
conversation in slack about this:
https://desikasgroupofawesome.slack.com/archives/C5HBZLSKX/p1643211197032300

Beyond this, make sure the box size, cosmology, etc. are what you set
them to in MUSIC.  Then, you should be in business to run!  This is an
example from one of Sidney's zooms::

  [desika.narayanan@login1 zooms]$ pwd
  /orange/narayanan/s.lower/simba/m25n256_dm/zooms
  [desika.narayanan@login1 zooms]$ more simba_ompi.job
  #!/bin/bash
  #SBATCH --job-name=r31_ml11
  #SBATCH --output=run_logs/run31_ml11.log
  #SBATCH --mem-per-cpu=3900
  #SBATCH --time=96:00:00
  #SBATCH --mail-user=s.lower@ufl.edu
  #SBATCH --mail-type=ALL
  #SBATCH --ntasks=512
  #SBATCH --ntasks-per-socket=8
  #SBATCH --distribution=cyclic:cyclic
  #SBATCH --cpus-per-task=1
  ##SBATCH --partition=hpg-default
  #SBATCH --account=narayanan
  #SBATCH --qos=narayanan-b
  ##SBATCH --account=astronomy-dept
  ##SBATCH --qos=astronomy-dept-b
  
  
  module purge
  module load intel/2018
  module load hdf5/1.10.1
  module load openmpi/3.1.2
  module load gsl/2.4
  module load fftw/2.1.5
  module load grackle
  
  export OMPI_MCA_pml="ucx"
  export OMPI_MCA_btl="^vader,tcp,openib"
  export OMPI_MCA_oob_tcp_listen_mode="listen_thread"
  
  DATADIR=$SLURM_SUBMIT_DIR
  cd $DATADIR/gizmo_simba_track_dust
  srun --mpi=pmix_v2  GIZMO $DATADIR/ml11_zoom_param_files/run31_halo0_ml11.param
  




Cosmological Simulations (Zoom-in)
============

Idealized Galaxy Simulations
============

First, all of the relevant files live on a private GitHub space at
https://github.com/dnarayanan/arepo_ics - be sure to ask if you don't
have permissions here.

Initial Conditions
-----------------

Make New Disk:

The first thing we'll need to do is run MakeNewDisk in order to make
an idealized galaxy disk IC.  You can find some examples on HiPerGator
here::

  /home/paul.torrey/InitialConditions/PhilsSpecificICs/MW
  /home/paul.torrey/InitialConditions/PhilsSpecificICs/SMC
  /home/paul.torrey/InitialConditions/PhilsSpecificICs/Sbc
  /home/paul.torrey/InitialConditions/PhilsSpecificICs/HiZ
  
(though note because of the size only the MW example is on GitHub).  You have to edit the resolution in main.c and then::

  make clean
  make

This is light weight code and can usually run from even a login node.  You can just run from something like::

  ./MakeHubbleType ./output/MW.dat

which produces your basic initial condition file.

**Adding Backgrounds for Arepo Simulations** If we were running a
gizmo simulation, we could stop right here.  Note, however, if we are
running an arepo simulation, this will require a background grid to be
added to the IC.  To do this, we use sbatch_makeIC.sh which is a job
script that calls arepo from the arepo_addbg directory.  Note, you'll
need a working executable for arepo in that directory.  There are
param files that are necessary for adding the background -- these
parameter files will have the params you'll use at simulation runtime,
and there are examples in::

  /blue/narayanan/desika.narayanan/MakeGalaxy/arepo_addbg

as well as the GitHub repo.  An example of this .sh file (for posterity) is::

  (pd4env_gcc) [desika.narayanan@login2 MakeGalaxy]$ more sbatch_makeIC.sh
  #!/bin/bash
  #SBATCH --job-name=makeIC
  #SBATCH --mail-type=ALL
  #SBATCH --mail-user=desika.narayanan@gmail.com
  #SBATCH --time=6:00:00
  #SBATCH --nodes=1
  #SBATCH --tasks-per-node=8
  #SBATCH --ntasks-per-socket=8
  #SBATCH --cpus-per-task=1
  #SBATCH --distribution=cyclic:cyclic
  #SBATCH --mem-per-cpu=8gb
  #SBATCH --partition=hpg2-compute
  #SBATCH --account=narayanan
  #SBATCH --qos=narayanan-b

  
  module purge
  #module load ddt/18.0.2
  module load intel/2018
  module load gsl
  module load openmpi/3.1.2
  module load hdf5
  #module load grackle
  
  DATADIR=$SLURM_SUBMIT_DIR
  
  export OMPI_MCA_pml="ucx"
  export OMPI_MCA_btl="^vader,tcp,openib"
  export OMPI_MCA_oob_tcp_listen_mode="listen_thread"
  
  srun --mpi=pmix_v2     ./arepo_addbg/Arepo   arepo_addbg/param_MW_ultra_lowres.txt 0        1> output_makeIC/OUTPUT  2> output_makeIC/ERROR



When you run this background addition, it
will automatically make a new file that has the appendate
"--with-grid.hdf5".  For example if your input file was called
"MW_lr.dat", your output file (assuming output type 3 is used) will be
called "MW_lr.dat-with-grid.hdf5"

**Further Modifying ICs for Dust**

We have one final step which is to move all type 2 and type 3 particles to type 4.  We do this with a script written by Qi Li called modifyIC.py -- there's an example on GitHub as well as at::

  /blue/narayanan/desika.narayanan/MakeGalaxy

Run this and we should have a new IC that is ready for Arepo as well!

Run Time
-----------------

You can find examples of the parameter files in the GitHub subdirectory idealized_repo_example or at::

  /blue/narayanan/desika.narayanan/arepo_runs/idealized/MW_ultra_lowres
