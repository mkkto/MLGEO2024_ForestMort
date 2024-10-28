## Predicting tree mortality for California and beyond

As climate change continues, the evaporative demand on plant life also increases [(Anderegg et al., 2019)](https://www.nature.com/articles/nclimate1635). This predisposes trees to die through a combination of processes [(Choat et al., 2018)](https://www.nature.com/articles/s41586-018-0240-x):

 - Carbon starvation
 - Hydraulic failure
 - Attack by boring insects

Mortality events have been documented worldwide, but are particularly severe in the southwestern US. This project has two goals:
 1. Improve annual predictions of the distribution of drought-induced tree mortality.
 2. Identify drivers of mortality over time to determine which mechanisms are most important.

## Setup
First, clone the repository
```
git clone https://github.com/UW-MLGEO/MLGEO2024_ForestMort
cd MLGEO2024_ForestMort
```
We depend on a variety of packages and not all of which are available on conda. We use conda to create the environment and pip to install packages. First, create the conda environment and install conda-specific packages.
```
conda env create --file=environment.yml
conda activate forest_mort
```
Install remaining packages with pip (this takes about 10 minutes on a fresh install).
```
pip install -r requirements.txt
```
Most of the data cleaning scripts use GDAL on the command line. GDAL is included in `environment.yml`, but if you find that you can't run GDAL commands check [this page](https://gdal.org/en/latest/api/python_bindings.html) for guidance on modifying your environment.

If you want to use any of the scripts that work with Earth Engine or `earthaccess`, you will have to set up accounts with the respective providers. **This is not necessary unless you want to recreate the steps we took to build the mortality datasets.** Once you have done so, do the following:
 - `import ee; ee.Initialize()` to link your Python session with your Earth Engine account.
 - Create a file named `.netrc` in your home directory. Add your `earthaccess` credentials to the file in the following format
```machine urs.earthdata.nasa.gov login <your_username> password <your_password>```
Now you should be able to authenticate with `earthacess.login(strategy="netrc")`.

For access to development data on GCS, you will have to also set up the Google Cloud SDK. You can have the SDK point to the conda environment we just created, or let it install the bundled Python. Follow the directions [here](https://cloud.google.com/sdk/docs/install) for your machine and then run the following two commands.
```
gcloud init
gcloud auth application-default login
```
Make sure you select the correct cloud project and Google account.

## Datasets
So far we have developed two forest mortality datasets, which we call ca_mort and west_mort. Both use [Aerial Detection Surveys](https://www.fs.usda.gov/science-technology/data-tools-products/fhp-mapping-reporting/detection-surveys) from the US Forest Service as the response variable. 

ca_mort was created with Earth Engine to reproduce the workflow in [Preisler et al. (2017)](https://www.sciencedirect.com/science/article/pii/S0378112717304772). The workflow is described in the following scripts/notebooks:
 - `scripts/ca_mort/clean_ca_ads_polygons.sh`
 - `notebooks/make_ads_images.ipynb`
 - `notebooks/make_tensors.ipynb`

The resulting dataset is at 4 km resolution and covers all ADS surveys in California from 1998 - 2018. The data are encoded as a CSV that is < 1 GB, but coordinate information is included so you can transform it into a spatial data structure. You can also download the data from a public cloud storage bucket. The URL and some visualization tricks are in `notebooks/get_drought_mortality.ipynb`.

west_mort is still under development, but is far enough along that we give some documentation details. **What follows is under active development and is not guaranteed to work!** This dataset is much larger than ca_mort. It covers all of the western US at 1 km resolution. We used an entirely open-source workflow to generate the dataset. Given the scale of data involved, the cleaning steps are spread across several scripts and notebooks.
 - `scripts/west_mort/download_data.sh` downloads all the input data to your machine. This will take up about 10 GB of disk space.
 - `scripts/west_mort/merge_ads_polygons.sh` combines ADS polygons from different USFS regions into one geodatabase.
 - `scripts/west_mort/burn_ads_polygons.py` converts ADS polygons into rasters.
 - `scripts/west_mort/burn_mtbs_polygons.py` converts polygons from the Monitoring Trends in Burn Severity dataset into rasters.
 - `scripts/west_mort/coarsen_genus_ba.py` resamples basal area rasters from the National Insect and Disease Risk Mapping program to match other rasters.
 - `scripts/west_mort/download_(daymet|terrain|vodca).py` all use the `earthaccess` library to download and summarize remote sensing data.
 - `notebooks/combine_netcdfs.ipynb` converts all the NetCDF files generated by the above scripts into windowed zarr arrays on cloud storage for use with a neural network.

## License
See file `LICENSE`. This repo uses the MIT license to support collaboration, and I strongly encourage you to reach out to me if you want to work on this problem!

