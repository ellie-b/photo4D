# Photo4D: open-source time-lapse photogrammetry 

Contributors by alphabetical orders:
- Simon Filhol (simon.filhol@geo.uio.no)
- Luc Girod (luc.girod@geo.uio.no)
- Alexis Perret (aperret2010@hotmail.fr)
- Guillaume Sutter (sutterguigui@gmail.com)

## Description

This project consists of an automated program to generate point cloud from time-lapse set of images from independent cameras. The software: 
1. sorts images by timestamps, 
2. assess the image quality based on lumincace and bluriness, 
3. identify automatically GCPs through the stacks of images, 
4. run Micmac to compute point clouds, and 
5. convert point cloud to rasters. (not implemented)

The project should be based on open-source libraries, for public release. 

## Reference

Filhol, S., Perret, A., Girod, L., Sutter, G., Schuler, T. V., and Burkhart, J. F.. ( 2019), Time‐lapse Photogrammetry of Distributed Snowdepth During Snowmelt. Water Resour. Res., 55. https://doi.org/10.1029/2018WR024530 

URL: https://agupubs.onlinelibrary.wiley.com/doi/10.1029/2018WR024530


## Installation
1. install the latest version of [micmac](https://micmac.ensg.eu/index.php/Install)

2. install python 3.6, and with anaconda, create a virtual environment with the following packages: 
     - opencv 
     - pandas 
     - matplotlib
     - lxml
     - pillow
     - [pyxif](https://github.com/zenwerk/Pyxif) (that needs to be downloaded from https://github.com/zenwerk/Pyxif)
     ```sh
     wget https://github.com/zenwerk/Pyxif/archive/master.zip
     unzip master.zip
     cd Pyxif-master
     mv LICENCE.txt LICENSE.txt   # As there is a typo in the License filename
     python setup.py install
     ```
     - [PDAL](https://pdal.io/)
     - json

 3. The package is available via Pypi

     ```python
     pip install photo4d
     ```

## Usage

### 1. prepare your environment: 
      - create a Python >= 3.6 virtual environment in which you install the required libraries (see above)
      - create a folder for the project with inside the project folder a folder called Images containing itself one folder per
      - Organize your photo with one folder per camera. For instance folder /cam1 constains all the images from Camera 1.
       camera
       
```bash
├── Project
    └── Images
         ├── Cam1
         ├── Cam2
         ├── Cam3
         └── Cam...
```


### 2. Use the Photo4d class to process the images through MicMac

Set the path correctly in the file MicmacApp/Class_photo4D.py, and follow these steps

```python

############################################################
## Part 1

import photo4d as p4d

# Create a new photo4d object by indicating the Project path
myproj = p4d.Photo4d(project_path="point to project folder /Project")

# Algorithm to sort images in triplets, and create the reference table with sets :date, valid set, image names
myproj.sort_picture()

# Algorithm to check picture quality (exposure and blurriness)
myproj.check_picture_quality()

############################################################
## Part 2: Estimate camera orientation

# Compute camera orientation using the timeSIFT method:
myproj.timeSIFT_orientation()

# Convert a text file containing the GCP coordinates to the proper format (.xml) for Micmac
myproj.prepare_gcp_files(path_to_GCP_file, file_format="N_X_Y_Z")

# Select a set to input GCPs
myproj.set_selected_set("DSC02728.JPG")

# Input GCPs in 3 steps
# first select 3 to five GCPs to pre-orient the images
myproj.pick_initial_gcps()

# Apply transformation based on the few GCPs previously picked
myproj.compute_transform()

# Pick additionnal GCPs, that are now pre-estimated
myproj.pick_all_gcps()

############################################################
## Part2, optional: pick GCPs on extre image set
## If you need to pick GCPs on another set of images, change selected set (this can be repeated n times):
#myproj.compute_transform()
#myproj.set_selected_set("DSC02871.JPG")
#myproj.pick_all_gcps()

# Compute final transform using all picked GCPs
myproj.compute_transform(doCampari=True)

## FUNCTION TO CHANGE FOR TIMESIFT
# myproj.create_mask() #To be finished

############################################################
## Part3: Compute point clouds

# Compute point cloud, correlation matrix, and depth matrix for each set of image
myproj.process_all_timesteps()

# Clean (remove) the temporary working direction
myproj.clean_up_tmp()

```

### 3. Process the point clouds with [PDAL](https://pdal.io/)

**Currently Under Development**

[PDAL](https://pdal.io/) is a python library to process point cloud. It has an extensive library of algorithms available, and here we wrapped a general method to filter and extract Digital Elevation Models (DEMs) from the point clouds derived in the previous step.

Micmac produces point clouds in the format `.ply`. The functions in the python class `pcl_process()` can convert, filter and crop the `.ply` point clouds and save them as `.las` files. Then the function `convert_all_pcl2dem()` will convert all point clouds stored in `my_pcl.las_pcl_flist` to DEMs. 

With the function `my_pcl.custom_pipeline()`, it is possible to build custom processing pipeline following the PDAL JSON syntax. This pipeline can then be executed by running the function `my_pcl.apply_custom_pipeline()`.

See the source file [Class_pcl_processing.py](./photo4d/Class_pcl_processing.py) for more details.

```python

# Create a pcl_class object, indication the path to the photo4d project
my_pcl = p4d.pcl_process(project_path="path_to_project_folder")

my_pcl.resolution = 1  # set the resolution of the final DEMs

# Set the bounding box the Region of Interest (ROI)
my_pcl.crop_xmin = 416100
my_pcl.crop_xmax = 416900
my_pcl.crop_ymin = 6715900
my_pcl.crop_ymax = 6716700
my_pcl.nodata = -9999

# add path og the .ply point cloud files to the python class
my_pcl.add_ply_pcl()

# filter the point clouds with pdal routine, and save resulting point clouds as .las file
my_pcl.filter_all_pcl()

# add path of the .las files
my_pcl.add_las_pcl()

# conver the .las point clouds to DEMs (geotiff)
my_pcl.convert_all_pcl2dem()

# Extract Value orthophoto from RGB 
my_pcl.extract_all_ortho_value()

```

After this section you have clean point clouds, as well as DEMs in GeoTiff ready!


## Ressources

- Micmac: http://micmac.ensg.eu/index.php/Accueil
- Image processing images: skimage, openCV, Pillow
- python package to read exif data: https://pip.pypa.io/en/latest/user_guide/

## Development

Message us to be added as a contributor, then if you can also modify the code to your own convenience with the following steps:

To work on a development version and keep using the latest change install it with the following

```shell
git clone git@github.com:ArcticSnow/photo4D.git
pip install -e [path2folder/photo4D]
```

and to upload latest change to Pypi.org, simply:

1. change the version number in the file ```photo4d/__version__.py```
2.  run from a terminal from the photo4D folder, given your $HOME/.pyc is correctly set:

```shell
python setup.py upload
```

