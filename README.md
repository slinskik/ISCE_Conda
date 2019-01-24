# ISCE_Conda
Notes for a working ISCE v2.1.0 install in a Conda environment

Following: https://github.com/scottyhq/isce_notes/tree/master/Ubuntu

1)	Get miniconda: 

wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh

bash Miniconda3-latest-Linux-x86_64.sh

2)	install packages (if missing):

sudo apt-get install libgmp-dev libmpfr-dev libmpc-dev libc6-dev-i386

3)	Get ISCE:

- Download latest from: https://winsar.unavco.org/isce.html

- Untar file

Following: http://earthdef.caltech.edu/boards/4/topics/1245?r=1417#message-1417

4)	Create two Python environments (following the naming convention defined in the README).

- Python 2.x environment called "isce" 

conda create --name isce python=2

- Python 3.x, called "isce3" 

conda create --name isce3 python=3

5)	Append the two conda environments to your environmental PATH.

export PATH=$HOME/miniconda3/envs/isce/bin:$HOME/miniconda3/envs/isce3/bin:$PATH

6)	Install the following packages into the "isce" conda environment (python 2.x).

conda remove --features mkl -n isce

conda install h5py -n isce

conda install scons -n isce

conda install -c andrewannex spiceypy=1.1.0 -n isce

7)	Install the following packages into the "isce3" conda environment (python 3.x)

conda remove --features mkl -n isce3

conda update --all -n isce3

conda install scipy -n isce3

conda install matplotlib -n isce3

conda install hdf5=1.8.15.1 -n isce3

conda install h5py -n isce3

conda install libgdal=2.0.0 -n isce3

conda install gdal=2.0.0 -n isce3

conda install krb5 -n isce3

conda install kealib=1.4.5 -n isce3

conda install -c andrewannex spiceypy=1.1.0 -n isce3

8)	Define the build environment. I uncompressed the required archive to my home directory under the name "isce-2.1.0"

export ISCE_BUILD_ROOT=$HOME/isce-2.1.0/build

export ISCE_INSTALL_ROOT=$HOME/isce

export SCONS_CONFIG_DIR=$HOME/.isce

mkdir $ISCE_BUILD_ROOT

mkdir $ISCE_INSTALL_ROOT

mkdir $SCONS_CONFIG_DIR

9)	Create a file called “SConfigISCE” file in SCONS_CONFIG_DIR that looks like this:

PRJ_SCONS_BUILD = $HOME/isce-2.1.0/build

PRJ_SCONS_INSTALL = $HOME/isce/isce

LIBPATH = $HOME/miniconda3/envs/isce3/lib /usr/lib/x86_64-linux-gnu

CPPPATH = $HOME/miniconda3/envs/isce3/include $HOME/miniconda3/envs/isce3/include/python3.5m /usr/include

FORTRANPATH = /usr/include

FORTRAN = /usr/bin/gfortran

CC = /usr/bin/gcc

CXX = /usr/bin/g++

MOTIFLIBPATH = /usr/lib/x86_64-linux-gnu

X11LIBPATH = /usr/lib/x86_64-linux-gnu

MOTIFINCPATH = /usr/include/Xm 

X11INCPATH =/usr/include/X11

10)	modify a few files to get the application to build correctly.

cd $HOME/isce-2.1.0

gedit ./SConscript

- Change:

env.PrependUnique(LIBS=['gdal'])

As follows:

- Resolve circular dependencies for gdal provided by Anaconda-Python...

env.PrependUnique(LIBS=['gdal', 'kea', 'hdf5', 'hdf5_hl', 'hdf5_cpp'])

- Include the following search path when linking against the conda environment, e.g. isce3.

env.AppendUnique(LINKFLAGS=['-Wl,-rpath,/home/kim/miniconda3/envs/isce3/lib'])

- Include system libraries

env.AppendUnique(LINKFLAGS=['-Wl,-rpath,/usr/x86_64-linux-gnu'])

Note: Replace “/home/kim/miniconda3/envs/isce3/lib" with the correct library path for the isce3 environment.

Appears to resolve the linker warnings generated in the config.log.

gedit ./contrib/mdx/src/SConscript

- Find line containing:

for i in range(envmdx['LIBS'].count('hdf5')): envmdx['LIBS'].remove('hdf5')

Append:

- Remove additional libraries required by Anaconda-Python

for i in range(envmdx['LIBS'].count('kea')): envmdx['LIBS'].remove('kea')

for i in range(envmdx['LIBS'].count('hdf5_cpp')): envmdx['LIBS'].remove('hdf5_cpp')

for i in range(envmdx['LIBS'].count('hdf5_hl')): envmdx['LIBS'].remove('hdf5_hl')

11)	 Build!

- From inside $HOME/isce-2.1.0

scons install

12)	Post installation modifications

Add to your bash rc file:

- Always import the conda environments

export PATH=$HOME/miniconda3/envs/isce/bin:$HOME/miniconda3/envs/isce3/bin:$PATH

- ISCI install path

export ISCE_INSTALL_ROOT=$HOME/isce

export PYTHONPATH=$ISCE_INSTALL_ROOT:$PYTHONPATH

- Import the ISCE path

export ISCE_HOME=$ISCE_INSTALL_ROOT/isce

export PATH=$ISCE_HOME/applications:$PATH
