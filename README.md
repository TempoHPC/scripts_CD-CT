# MONAN - Model for Ocean-laNd-Atmosphere PredictioN

### *Laptop version for running MONAN in personal computers*

This procedure aims to create a version for testing MONAN using laptops, running MONAN at low resolution using GFS.

Is Section *Preparing to run the model*, it install libraries, download data, the MONAN model and the scripts to run.
In Section *Run the MONAN Model*, it shows how to run MONAN model using the scripts.

This version is based in 0.2.1 scripts_CD-CT version for Egeon

Preparing to run the model
==========================

This steps install libraries, download data, the MONAN model and the scripts to run.

**1. First of all, clone the suite scripts that you will use:**

~~~
 git clone https://github.com/monanadmin/scripts_CD-CT.git
~~~
you will get this directories:
~~~
datain/namelists
scripts
~~~

- The `datain/namelists` directory contains all versioned namelists needded for run and compile all phases of model;
- The `scripts` directory is the most important folder that contains all the scripts that you will need to install, compile, run, and produce produtcs of the A-MONAN model.


Switch to temporary correct branch "mylocal_version_0.2.1" and enter the scripts directory.
~~~
git checkout mylocal_version_0.2.1
cd mylocal_version_0.2.1
~~~


**2. Install spack:**

Install spack, that is a dependency manager for installing libraries. It installs libraries locally, without damaging your system libraries.
~~~
./install_spack.bash
~~~

**3. Install the MPAS dependencies and other packages using spack:**

*IMPORTANT: Execute the command below again in case you stoped the sequence of commands from here, then continue executing the commands, starting from the last command (inclusive) you ran )*
~~~
source spack/env.sh
~~~

The command below will show the gcc compiler you have. Check its version in the results of this command and use it instead of the version 9.4.0, just in case there are diferences. For example, use the command returned  gcc@9.4.0. So use this version in all the commands below.


~~~
spack compiler find
~~~

Then use the right version of the compiler in mpas-model and wps instalation below. Example, if the command above returns `gcc@9.4.0  gcc@8.4.0  gcc@7.5.0`, use the most recent version `gcc@9.4.0`, i.e. below.

~~~
spack clean --all
spack env create myenv
spack env activate myenv
spack add mpas-model@7.3%gcc@9.4.0 ^parallelio+pnetcdf
spack add wps@4.5%gcc@9.4.0
spack add metis@5.1.0%gcc@9.4.0
spack add python@3.9.13%gcc@9.4.0
# spack add cdo%gcc@9.4.0 - error - needed fixes
spack concretize
spack install

# needed until fix for installation using spack
sudo apt install cdo  
~~~

Some versions of packages could be different using different versions of the compiler. The versions above, i.e. metis version 5.1.0, are compliant with the gcc@9.4.0. If you have problems when installing, for example "package not found", suggesting another version, you can use the version suggested or simply do not use the version, i.e. "metis%gcc@9.4.0"

**4. Configure the PNETCDF and NETCDF environment vars**

execute below to find the correct path ...
~~~
spack find -p netcdf-c netcdf-fortran parallel-netcdf
~~~ 

... and modify the lines below in scripts_CD-CT/scripts/setenv.bash script, by updating the paths shown in the command above. I.e:

Libraries paths:
~~~
  export NETCDF=/home/user/repo/scripts_CD-CT/scripts/spack/opt/spack/linux-ubuntu22.04-haswell/gcc-11.4.0/netcdf-c-4.9.2-mrzvvju2bmuw76i6r6byrx2kn6ygyhqg
  export PNETCDF=/home/user/repo/scripts_CD-CT/scripts/spack/opt/spack/linux-ubuntu22.04-haswell/gcc-11.4.0/parallel-netcdf-1.12.3-dgphmay73qspxoi3pkqkpzocf4gb2oq3
~~~


**5. Download the data pack into scripts_CD-CT/datain directory:**

This are fixed data and must be downloaded only once. First check if you have available space (at least 120 GB)


Enter the datain directory:

~~~
cd scripts_CD-CT/datain
~~~

Run the lines below to get data. This step takes about 20 minutes to finish.
~~~  
  wget https://ftp.cptec.inpe.br/pesquisa/dmdcc/volatil/Renato/scripts_CD-CT_datain.tgz
  wget https://ftp.cptec.inpe.br/pesquisa/dmdcc/volatil/Renato/gfs.t00z.pgrb2.0p25.f000.2024080800.grib2
~~~

Run below to Untar fixed data. This step takes about 15 minutes to finish.
~~~
tar -xzvf scripts_CD-CT_datain.tgz
~~~

The following directories will be generated by the command above:
~~~
  scripts_CD-CT/datain/fixed
  scripts_CD-CT/datain/WPS_GEOG
~~~

Now move gfs file to the data directory inside datain:
~~~
mkdir 2024080800
mv gfs.t00z.pgrb2.0p25.f000.2024080800.grib2 2024080800
~~~

**4.1 Optional**

For other CI dates, you can get by:
~~~
  wget https://ftp.ncep.noaa.gov/data/nccf/com/gfs/prod/gfs.20240819/00/atmos/gfs.t00z.pgrb2.0p25.f000
~~~

Rename it to the appropriate GFS name that the script is waiting for:
~~~
mv gfs.t00z.pgrb2.0p25.f000 gfs.t00z.pgrb2.0p25.f000.2024081900.grib2
mkdir 2024081900
mv gfs.t00z.pgrb2.0p25.f000.2024081900.grib2 2024081900
~~~


Run the MONAN Model
===================


You will need to execute only 6 steps scripts, so you can run the Atmospheric MONAN Model:


**1. Install the model:**

If your intent is just test the model with original code, execute:
~~~
./1.install_monan.bash https://github.com/monanadmin/MONAN-Model
~~~

If you intend to develop, first you need to get a **fork repository** in your github account of a MONAN oficial repo: `https://github.com/monanadmin/MONAN-Model`. Attention! Uncheck "Copy the main branch only" in the fork creation step to copy all branches. 

The you can install the model in your work directory by running:

~~~
./1.install_monan.bash <https://github.com/MYUSER/MONAN-Model-My-Fork.git> <OPTIONAL_tag_or_branch_name_MONAN-Model-My-Fork> <OPTIONAL_tag_or_branch_namer_Convert-MPAS>
~~~

Default values:
~~~
<OPTIONAL_tag_or_branch_name_MONAN-Model> = "1.0.0"
<OPTIONAL_tag_or_branch_name_Convert-MPAS> = "1.0.0"
~~~

The command 1.install_monan.bash will create the diretories structure:
~~~
scripts_CD-CT/
       scripts
       sources
       execs
       datain
       dataout
~~~

Where:
- `scripts` folder will contain all scripts produced to run all steps of the model;
- `sources` folder will contain all codes of any processes that uses compiled programming languages, such as MONAN model sources, convert_mpas sources, etc.
- `execs` folder will contain all the executables needed;
- `datain` folder will contain all the input data that the model need to run;
- `dataout` folder will contain all the output files generated of running of the MONAN, such as:
     - `dataout\Pre\<YYYYMMDDHH>` will contain all the output files from the pre-processing phase, mostly are all the initial condition for run the MONAN;
     - `dataout\Model\<YYYYMMDDHH>` will contain all the output files from the MONAN model;
     - `dataout\Post\<YYYYMMDDHH>` will contain all the output files from the post-processing phase of the MONAN;
     - `dataout\Prods\<YYYYMMDDHH>` will contain all the output files from the products generated, graphics, derivated variables, peace of domain, etc.

After running the first step, it will clone the MONAN model from your fork repo in a `source` diretory.


**2. Prepare the Initial Conditions for the model:**

Just run the second script as follows:

~~~
2.pre_processing.bash EXP_NAME RESOLUTION LABELI FCST

EXP_NAME    :: Forcing: GFS
            :: Others options to be added later...
RESOLUTION  :: number of points in resolution model grid, e.g: 1024002  (24 km)
LABELI      :: Initial date YYYYMMDDHH, e.g.: 2024010100
FCST        :: Forecast hours, e.g.: 24 or 36, etc.

480Km resolution 24 hour forcast example:

./2.pre_processing.bash GFS 2562 2024080800 24
~~~

**3. Run the model:**

Execute the 3rd step script:

~~~
3.run_model.bash EXP_NAME RESOLUTION LABELI FCST

EXP_NAME    :: Forcing: GFS
            :: Others options to be added later...
RESOLUTION  :: number of points in resolution model grid, e.g: 1024002  (24 km)
LABELI      :: Initial date YYYYMMDDHH, e.g.: 2024010100
FCST        :: Forecast hours, e.g.: 24 or 36, etc.

24 hour forcast example:
./3.run_model.bash GFS 1024002 2024010100 24
~~~

**4. Run the post-processing model:**

Execute the 4th step script:

~~~
4.run_post.bash EXP_NAME RESOLUTION LABELI FCST

EXP_NAME    :: Forcing: GFS
            :: Others options to be added later...
RESOLUTION  :: number of points in resolution model grid, e.g: 1024002  (24 km)
LABELI      :: Initial date YYYYMMDDHH, e.g.: 2024010100
FCST        :: Forecast hours, e.g.: 24 or 36, etc.

24 hour forcast example:
./4.run_post.bash GFS 1024002 2024010100 24
~~~
