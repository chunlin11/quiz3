[![build status](https://gitlab.in2p3.fr/stas/MLDC/badges/master/build.svg)](https://gitlab.in2p3.fr/stas/MLDC/commits/master)
[![coverage report](https://gitlab.in2p3.fr/stas/MLDC/badges/master/coverage.svg)](https://gitlab.in2p3.fr/stas/MLDC/commits/master)

development branch : [![build status](https://gitlab.in2p3.fr/stas/MLDC/badges/development/build.svg)](https://gitlab.in2p3.fr/stas/MLDC/commits/development)

# Mock LISA Data Challenge Project


## Welcome to the project.

The ultimate goal is to build the LISA data analysis pipeline completely integrated into the end-to-end simulator.
Please refer to wiki for LDC definition.

The last version of the documentation in pdf is at [here  (https://gitlab.in2p3.fr/stas/MLDC/builds/artifacts/master/download?job=doc)](https://gitlab.in2p3.fr/stas/MLDC/builds/artifacts/master/download?job=doc) for the master branch.

----

## Installation

### Jupyter Docker

There is a ready-to-use docker image containing an installation of all the LDC tools and a jupyter interface.
To use the image you need [Docker](https://www.docker.com/) installed on your system.

- In a terminal run the 3 following commands:
```shell
    docker login gitlab-registry.in2p3.fr
    docker pull gitlab-registry.in2p3.fr/elisadpc/docker/ldc_jupyter:master
    docker run -it -p 8888:8888 -v PathMyLocalDir:/home/jovyan/work gitlab-registry.in2p3.fr/elisadpc/docker/ldc_jupyter:master
```
You should obtain something like :
```
Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://localhost:8888/?token=07f85fd96341705e1ecb5ce584b81582bacf0086638efd9d
```

- Follow the instruction and copy the URL in your browser. it should display the jupyter home page with the directory `work`. This is the directory shared with your local directory `PathMyLocalDir`.

- You can now use ipython notebooks (``.ipynb`). You can find some examples in the directory `notebooks` of this git.



### Docker for user
There is a ready-to-use docker image containing an installation of all the LDC tools.
This image is automatically updated after each push passing the tests.
To use the image you need [Docker](https://www.docker.com/) installed on your system.
Once installed Docker, you can use it by entering following commands in your terminal:

```shell
    docker login gitlab-registry.in2p3.fr
    docker pull gitlab-registry.in2p3.fr/stas/mldc:master
    docker run -it -v PathMyLocalDir:/workspace gitlab-registry.in2p3.fr/stas/mldc:master
```

- `PathMyLocalDir` is the path to a local directory on your computer that is mirrored with the directory `workspace` on the docker instance. It is use to exchange files between docker and the rest of your computer.
- `master` is the branch used to build the image

If everything went well, the last line of your terminal should start with `bash-4.2#`.
If it's the case you are in the docker machine contianing the LDC tools.
You are in are in the `/workspace` directory on docker which is a mirror of your local directory `PathMyLocalDir`:

```shell
   bash-4.2# pwd
   /workspace
   bash-4.2#
```




### Docker for developer
There is docker image containing all the necessary packages to install and to run the LDC tools.
To use the image you need [Docker](https://www.docker.com/) installed on your system.

Once installed Docker, make sure you have `git` installed and then:
1. Clone the LDC git repository somewhere on your computer (`MYDIR/LDC`):

```shell
    cd MYDIR/LDC
    git clone git@gitlab.in2p3.fr:stas/MLDC.git
```

2. Start docker  

```shell
    docker login gitlab-registry.in2p3.fr
    docker pull gitlab-registry.in2p3.fr/elisadpc/docker/ldc_env:master
    docker run -it -v /PATH_TO_DIR_WITH_LDC:/workspace gitlab-registry.in2p3.fr/elisadpc/docker/ldc_env:master
```

The last line of your terminal should start with `workspace#`: your terminal is in the docker instance which a
standard unix environment (CentOS) and containing all the tools necessary for the LDC.
The directory `workspace` on the docker instance is a mirror of your local directory `/PATH_TO_DIR_WITH_LDC` which should be the directory containing the `LDC` directory cloned from git. If you type `ls` you should see the `LDC` directory.

3. Install the LDC tools and run the tests:

```shell
    cd MLDC
    python3 master_install.py
    export PYTHONPATH=/workspace/LDC/software/LDCpipeline/scripts/:$PYTHONPATH
    export PATH=/workspace/LDC/software/LDCpipeline/scripts/:$PATH
    python3 -m unittest discover
```
The two `export` lines are needed for indicating where are the function that
some script are calling and that are tested.
`/workspace/LDC` is the path to the LDC directory on the docker machine.

Now, you can make your change on the LDC scripts,etc and run the tools in the terminal


### Manual installation

#### How to
1. Make sure you have all the dependencies (see next section) installed.
2. Clone the repository:

```shell
    git clone git@gitlab.in2p3.fr:stas/MLDC.git
```

3. Run the `master_install`:

```shell
    cd MLDC
    python3 master_install.py --prefix=WhereToInstall
```

#### Dependencies

You will need to install:
- `c++` compiler
- `python3`
- `gsl`
- `fftw`
- `cython`
- `swig`
- [`LISACode`][https://gitlab.in2p3.fr/elisadpc/LISACode]
We also recommend installation of anaconda.  


#### Issues, problems
Use the issues section.

Any questions: mail to [stas@apc.in2p3.fr](mailto:stas@apc.in2p3.fr)


----
## How to use it ?

We assume here you start from the docker image (see section *Docker for user*)

### Run the pipeline to generate data

If you are using "docker for user" ( gitlab-registry.in2p3.fr/stas/mldc:master ),
you have all scripts corresponding to this section in `/codes/LDC/examples/README/DataGeneration`.
You can just copy them in the worspace and play !

```shell
cp -r /codes/LDC/examples/scripts_README/DataGeneration /workspace/.
```

<!-- start DataGeneration -->

#### Generate source list

- Start a configuration file. In the example, we copy `Ref_Param.txt` from `examples` and adapt the path of catalogues:  
```shell
cp /codes/LDC/examples/Ref_Param.txt  Param.txt
```

- Choose the sources and produce the `hdf5`
```shell
ChooseSources.py --paramFile=Param.txt --filename=MySim_Param.hdf5 --seed=12345
```

- You can check the parameters using `LISADhdf5` python3 class. Start `ipython3` then:
```python
# CheckSources
from LISAhdf5 import *
f = LISAhdf5('MySim_Param.hdf5')
print("Number of sources = ", f.getSourcesNum())
print("Name of sources = ", f.getSourcesName())
p = f.getSourceParameters('MBHB-0')
p.display()
```

#### Compute h+,hx
- Generate h+, hx:
```shell
Compute_hphc.py MySim_Param.hdf5
```
- To check the waveform. Start `ipython3` then:

```python
# CheckWaveform
from LISAhdf5 import *
import matplotlib.pyplot as plt
LH = LISAhdf5('MySim_Param.hdf5')
thphc = LH.getSourceHpHc('MBHB-0')
plt.plot(thphc[:,0],thphc[:,1])
plt.xlabel('Time(s)')
plt.ylabel('h+')
plt.savefig('hphc_MBHB0.png')
```

#### Configure Instrument and Noises
- Configure instrument:
```shell
ConfigureInstrument.py --duration=31457265 MySim_Param.hdf5
```
Duration is 1 year instead of 4 years for default. See `ConfigureInstrument.py --help` for more options (timestep, duration, TDI, orbits, ...). For example, if you want to use the MLDC orbits, add the option `--orbits=MLDC_Orbits`.

- Configure noises:
```shell
ConfigureNoises.py MySim_Param.hdf5
```

- Check content of hdf5:
```shell
LISAh5_display.py MySim_Param.hdf5
```


#### Run simulation
- Run simulation using LISACode2:
```shell
RunSimuLC2.py MySim_Param.hdf5
```

- To check the TDI. Start `ipython3` then:
```python
# CheckTDI
from LISAhdf5 import *
import matplotlib.pyplot as plt
LH = LISAhdf5('MySim_Param.hdf5')
tTDI = LH.getPreProcessTDI()
plt.plot(tTDI[:,0],tTDI[:,1])
plt.xlabel('Time(s)')
plt.ylabel('TDI X')
plt.savefig('TDI_X.png')
```

- To extract the TDI from hdf5 to ascci file:
```shell
LISAh5_tdi2ascii.py MySim_Param.hdf5 MySim_Param_TDI.txt
```

<!-- end DataGeneration -->


### Manipulate `LISAhdf5` files

- General tool to manipulate `hdf5` file [HDFView](https://support.hdfgroup.org/products/java/hdfview/)

- Display `LISAhdf5`
```shell
LISAh5_display.py MySim_Param.hdf5
```

- Edit `LISAhdf5`
```shell
LISAh5_edit.py MySim_Param.hdf5
```

- Extract TDI output in `LISAhdf5` in ASCII file:
```shell
LISAh5_tdi2ascii.py MySim_Param.hdf5 MySim_Param_TDI.txt
```

- Plot PSD of the TDI output from `LISAhdf5` (for more options see `psd.py -h`):
```shell
psd.py MySim_Param.hdf5
```

### Run the pipeline to reproduce LDC data

Assuming you are running the docker for user, copy the datasets directory in `workspace` (since it's the share directory you will have the data on your local machine)
```shell
cp -r /codes/LDC/datasets .
```

#### Radler MBHB

```shell
datasets/Radler/MBHB/
ipython3 -- ProduceMBHB_Radler.py  MBHB_RadlerTest_seed1911.hdf5 --seed=1911 --verbose
```

It  should have produce in the directory `datasets/Radler/MBHB/`:
- `MBHB_RadlerTest_seed1911.hdf5`
- `MBHB_RadlerTest_seed1911_FD_NoNoise.hdf5`
- `MBHB_RadlerTest_seed1911_TD_NoNoise.hdf5`
- `MBHB_RadlerTest_seed1911_FD.hdf5`
- `MBHB_RadlerTest_seed1911_TD.hdf5`
- `TestSignalsTDandFD.pdf`
-  


----

## For developers

### Git branches
In order to keep the `master` branch clean and tested, do not commit on the `master` branch and use merge request:
1. Go to `develop` branch:
```shell
    git checkout development
```

2. Commit and push your changes

3. On the web interface https://gitlab.in2p3.fr/stas/MLDC :
   1. Click `Merge Requests`
   2. Click `New Merge Request`
   3. `Select source branch` > `develop`
   4. Click `Compare branches and continue`
   5. Write `Title` and `Description` if you want, ...
   6. Click `Submit Merge Request`
   7. If you want to accept it direclty, click `Merge ...`


### Tests
The tests are in the directory `tests`.
Typically, you have one test file per script or class.

#### Python
We use the module `unittest` (more information at https://docs.python.org/3/library/unittest.html ). You need to create a class derived from `unittest.TestCase`. Example:
```python
# Unitests
import os
import unittest
from LISAhdf5 import LISAhdf5,ParsUnits

class Test_LISAhdf5(unittest.TestCase):
    """Test `LISAhdf5` class."""

    def test_getSourcesNum(self):
        """Test 'getSourcesNum' method of `LISAhdf5` """
        print "\n===== Test_LISAhdf5:test_getSourcesNum ====="
        LH = LISAhdf5('tests/ForTest_LISAhdf5.hdf5')
        self.assertEqual(LH.getSourcesNum(),1)
```

## Frequently Asked Questions

### Docker jupyter

#### Error starting userland proxy
If you get an error message ending with "Error starting userland proxy: 
listen tcp 0.0.0.0:8888: bind: address already in use." (Linux), you are 
probably already running a Jupyter notebook (or some other local web app) that 
conflicts with LISA's docker one.

To solve this, just change the external port in the binding, i.e. change the 
`-p` option of `docker run` to `-p [e.g. 8880]:8888`, and open the LISA docker 
Jupyter notebook in a browser with `http://127.0.0.1:[new port that you put in -p]`. 
Jupyter will ask for a token that you can get from the last line printed by docker, 
which should end with `token=[your token]` (if that token does not work, make 
sure that you are using the same port in the left field of `docker run -p` and in the browser address). 

#### Development good practices
A series of wiki entries has been written with some good practices about Python development.
You can find them [here](https://gitlab.in2p3.fr/stas/MLDC/wikis/good_dev_practices)
