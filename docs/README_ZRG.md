# Installation Process of ISCE and Mintpy

The following steps are to be followed for the installation of conda version of ISCE-2 and developer version of MintPy. The steps mostly come from [this](https://github.com/yunjunz/conda-envs) original installation guide.
## Step 1: Installation of Mamba / Conda

Install Python environment necessary for ISCE.

```bash
mkdir -p ~/tools #makes a folder "tools" inside your home directory
cd ~/tools #changes the current working directory to "tools"
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh #downloads the latest release of miniforge 3 installer for x86_64 architecture Linux environment.
bash Miniforge3-Linux-x86_64.sh -b -p ~/tools/miniforge #runs the installer in batch mode (-b) and installs into the path (-p) "~/tools/miniforge" 
~/tools/miniforge/bin/mamba init bash #initialize mamba environment for the current shell. This will setup the environment variables in the system and will automatically activate mamba when a new terminal session begins.
```

Close the terminal window and restart it for changes to take effect. If you see `(base)` in front of your username in your terminal session after restart, for example : `(base) netid@radar1:~` , it indicates that mamba has been successfully initialized.

After mamba has been initialized, install some prerequisite packages like `wget`,`git`, `tree` and `numpy`.

```bash
mamba install wget git tree numpy --yes #installs the prerequisite command and automatically answers "yes" to any prompt that might come up during installation.
```

## Step 2: Installation of ISCE-2 (Conda version) and MintPy (Developer version)

### a. Download source code 

```bash
cd ~/tools
mkdir isce2; cd isce2 #creates a new folder isce2 inside "tools" and changes the current working directory to this folder.
mkdir build install src; cd src #makes three new folders - build, install and src inside isce2, and changes the current working directory to src.
git clone https://github.com/isce-framework/isce2.git #clone isce2 from isce-framework repository inside the "src" folder.

cd ~/tools #change current working directory to the "tools" folder
git clone https://github.com/insarlab/MintPy.git #clones MintPy inside "tools" folder
git clone https://github.com/insarlab/PyAPS.git #clones PyAPS inside "tools" folder
git clone https://github.com/insarlab/PySolid.git #clones PySolid inside "tools" folder
git clone https://github.com/yunjunz/conda-envs.git #clones conda-envs installation guide from Dr. Yunjun's repository inside "tools" folder

mkdir -p ~/tools/utils; cd ~/tools/utils #creates a new folder "utils" inside "tools" and changes the current working directory to utils.
git clone https://gitlab.com/yunjunz/SSARA.git #clones SSARA into the "utils" folder
git clone https://github.com/yunjunz/sardem.git #clones sardem into the "utils" folder
```

### b. Create a new conda environment "insar" and install ISCE-2, mintpy and their dependencies

```bash
# create a new environment "insar" and activate it
conda create --name insar --yes
conda activate insar

# install dependencies and isce2
cd ~/tools #changes current working directory to tools
mamba install --file conda-envs/insar/requirements.txt --file MintPy/requirements.txt isce2">=2.6.3" --yes #installs all requirements for isce2 and MintPy. Also specifies that the version of isce2 must be >=2.6.3. Additionally, also automatically answers yes to any prompt that might come up during installation.

# install MintPy in editable mode
python -m pip install -e MintPy # "-m" allows to run python modules as scripts, whereas "-e" allows you to install the package in editable mode such that all changes in the source will be reflected immediately in the package.
# [for developers] overwrite PyAPS and PySolid installation from conda to the editable mode
python -m pip install -e PyAPS
export SETUPTOOLS_ENABLE_FEATURES="legacy-editable" #turns on the "legacy editable" feature that allows packages to run in editable mode.
python -m pip install -e PySolid

# install dependencies not available from conda
ln -s ${CONDA_PREFIX}/bin/cython ${CONDA_PREFIX}/bin/cython3 #creates a symbolic link so that anytime the command "cython3" is executed, it actually runs the default "cython" executable in your current conda environment.
python -m pip install ipynb     # import functions from *.ipynb files
python -m pip install jupyter_nbextensions_configurator #might give rise to an error for now. See below
python -m pip install -e utils/sardem #install sardem in editable mode
```

> [!WARNING]
> When executing the command `python -m pip install jupyter_nbextensions_configurator` , it might give rise to a dependency conflict that requires a package `pdflatex 0.1.3` to have another package `attrs<19.0,>=18.2`. If you manually try to change the version of `attrs` to `>=18.2`, this will then give rise to dependency conflict with other packages like `jsonschema`, `referencing` and `fiona`.

>[!NOTE]
> **Solution 1:** As `jupyter_nbextensions_configurator` is not a critical requirement, the error can be ignored for now and we can move on to the next step.
> 
>**Solution 2:** Putting all dependencies from conda-forge and pip into one `environment.yml` file and installing and managing the packages from that file.

For the sake of reducing complexities, we move on with solution 1, ignoring the error for now.
### c. Setup

Create an alias `load_insar` in `~/.bash_profile` file for easy activation:

```bash
alias load_insar='conda activate insar; source ~/tools/conda-envs/insar/config.rc'
```

Creating this alias will enable you to activate the environment by just executing `load_insar` in your terminal.

<div style="page-break-before: always; visibility: hidden;"></div>

>[!Tip]
>If you do not want to execute the line manually every-time you restart your shell session, edit your `~/.bashrc` file by executing `nano ~/.bashrc` command or a similar command, add the line `alias load_insar='conda activate insar; source ~/tools/conda-envs/insar/config.rc'` and save. 

### d. Testing the installation

If the following commands run successfully, it indicates that the installation was successful

```bash
load_insar               # activate the insar conda environment
topsApp.py -h            # test ISCE-2 installation
smallbaselineApp.py -h   # test MintPy installation
```

## Potential Issues

### No "tmp" directory

>[!WARNING] 
>During the installation, you might encounter a filesystem error with the message `filesystem error: temp_directory_path: No such file or directory [/usr/tmp]`. This error most likely comes from the fact that either the folder `/usr/tmp/` does not exist or you don't have access to it.

>[!NOTE]
>This can be solved by changing the `TMPDIR` environment variable by specifying path to a new temporary directory that is accessible to you. You can do this by executing `export TMPDIR=/path/to/new/tmpdir` command. An example `/path/to/new/tmpdir` could be `/home/your_netid/tmp`.
## References

1. Yunjunz. (n.d.). _conda-envs_. GitHub. https://github.com/yunjunz/conda-envs
