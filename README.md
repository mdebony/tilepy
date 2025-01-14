# Tilepy

## Installation

We clone the repo, create an environment to work, activate the environment and install the package.

```python
git clone git@github.com:astro-transients/tilepy.git
cd tilepy
conda env create -n tilepyenv -f environment.yml
conda activate tilepyenv
python -m pip install .
```

If you prefer to avoid conda and use a virtual environment with your favorite python version, use the following sequence:
```
python -m venv tilepy_venv
source tilepy_venv/bin/activate
pip3 install --upgrade pip
python -m pip install -r requirements.txt
```

Requirements of the installation: 

- The current version of the package **only** runs with `python>=3.9`. Python 3.9 is recommended. Careful as well with the versions of matplotlib and healpy, they should be the ones explicited in the requirements.yml, otherwise there will be conflicts between them when plotting skymaps.  
- Note that by creating the env from the environment.yml, the libraries and versions needed will be installed authomatically.
- Note that everytime we made changes to the package, you should update the installation of the package doing ```pip install .``` in the folder where the setup.py is. The changes will be only applied to the env where you are working. 
- The package relies on 'curl' to download the localisaton map of the multi-messenger events.

In the case you are working in CC-Lyon, the easiest solution is to do```ccenv conda ``` and then follow the instructions given above. 

If you have any problem with the installation of the package, please drop an email to astro.tilepy@gmail.com. 
## Description

Package including functions to perform GW follow-up scheduling and simulations in IACTS. The package can be found in the folder tilepy, which contains the following folders: 
 
- tilepy: Folder including the package
    - tilepy.tools: Includes several scripts that have been used so far for different aims related to visualization and catalog cleaning 
    - tilepy.include: The main functions used by the two main scripts are in this folder. It includes the Pointing Tools specifically for CTA (the others are in GWHESSPointing tool which is imported by GWCTAPointingTools), the CTA observation scheduler, simulation tools and analysis tools (both using gammapy)
    - tilepy.dataset: This is a folder where we recomend to put files as finals2000.all. You will see it empty. 

- relics: Old scripts that may be useful in the future

- examples: Example on how to use tilepy:
    - obsCases: plots and scripts used to produced the results presented in an incoming paper. It contains a variety of astrophysical cases and observatory configutations. 
    - obsLauncher: Jupyter notebooks and .py to run observation schedulings for 1 telescope and N telescopes. We recommend to use the Jupyter notebooks as these are more comprehensive, specially with the inputs given. 
    - obsConfig: three examples of configuration files, used in the notebooks to run the scripts. The format is the following: 
       - [observatory]
          - name: name of the observatory (it is not critical, you can use any name) 
          - lat: lat coordinates of the observatory 
          - lon: lon coordinates of the observatory 
          - height: height of the observatory 

        - [visibility]
          - sunDown: altitude of the Sun in deg to define darkness conditions (for astronomic darkness sunDown= -18)
          - moonDown: altitude of the Moon in deg to define darkness conditions (for astronomic darkness moonDown= -0.5)
          - moonGrey: altitude of the Moon in deg to define greyness conditions
          - moonPhase: phase of the Moon to define greyness conditions
          - minMoonSourceSeparation: minimum separation Source-Moon in deg to define greyness conditions
          - maxMoonSourceSeparation: max separation Source-Moon in deg to define greyness conditions

        - [operations]
          - maxZenith: max zenith angle which will be considered as accesible sky
          - FOV: radius of the circular FoV defining the tiles
          - maxRuns: maximum number of tiles that will be scheduled
          - maxNights: total number of nights considered
          - duration: standard exposure per tile
          - minDuration: minimal duration of tile if standard exposure is not available (e.g. at the end of the night)
          - useGreytime: flag to schedule greyness observations in addition to darkness

        - [tiling]
          - online: tbd
          - minimumProbCutforCatalogue: only galaxies that have probabilities higher than 'minimumProbCutforCatalogue x (GW x galaxy)_max' participate in the scheduling calculation
          - minProbcut:  minimal probability covered per tile to schedule observation
          - distCut: distance cut to define the mandatory use of 2D strategy (coming from galaxy catalogue completeness)
          - doPlot: produce detailed plots of the scheduling
          - secondRound: consider two maps for scheduling 
          - zenithWeighting: weight on probability that would be applied to prioritize coordinates that have lower zenith angle values. Step size is 5 deg in zenith (0.75 is a reasonable value)
          - percentageMOC: percentage of the sky localization region that will be considered to compute the MOC
          - reducedNside: nside of the low-resolution skymap used as a grid to speed up the computation
          - HRnside: nside of the high-resolution skymap used to compute the covered probability
          - mangrove: flag to use the mangrove method of weighting by the mass of the host galaxy

## Creation of the reduced galaxy file

For using the 3D algorithm tilepy need to have access to a galaxy catalog. Currently, the only supported catalog is GLADE+. You'll find the download link on this webpage : https://glade.elte.hu.
To prepare it for usage by tilepy we provide a the `ConvertGalaxyCatalog.py` script that converts the original catalog into a hdf5 file compatible with tilepy. The script is located in the `tilepy/tools` repository.

Example: use the script to keep only galaxies that are within 500 Mpc (recommended). It reads the downloaded `GLADE+.txt` file and creates the output file `converted_GLADE.h5` which is the one that should be used with tilepy.

```bash
python ConvertGalaxyCatalog.py --input GLADE+.txt --output converted_GLADE.h5 --max-luminosity-distance 500
```

## Issue with Daily Earth Orientation Parameters Solutions file (finals2000A)

Astropy requires a recent reference file to compute correctly the coordinates. This file is in general downloaded automatically by Astropy but if you are offline, you will be able to run tilepy if you do the following fix in a global variable.
First you need to download the file, it is available through several sources, IERS, OBSPM, NASA, USNO, .... For a fully offline installation, try to update the file every few month.

You need then to modify your script calling tilepy by adding the following lines before importing tilepy. Adapt the path to the file (here pathToYourReferencefile) in order that the system is able to find the file and loaded correctly.

```python
import os
from astropy.utils import iers
iers_file = os.path.join(os.path.abspath(
    os.path.dirname(__file__)), pathToYourReferencefile)
iers.IERS.iers_table = iers.IERS_A.open(iers_file)
```

## Help
If any problem is found, please open a new issue in the project main page to document it. You can of course also directly create a Pull Request with new features.

Otherwise you can also contact us at astro.tilepy@gmail.com.
