# FITTING WITH BAND DETECTION LIMITS

The versions of `likelihood.py` and `starmodel.py` here are meant to replace those in the isochrones package with minimal changes, so that normal use is not changed. The purpose of these changes is to allow the user to account for the sensitivity of filters when passing band magnitudes as initial values in a `StarModel` for pymultinest fitting. For example, 

```
from isochrones import mist, SingleStarModel
init_dict = {WFC3_HST_F336W_mag: (29, 2)} # (VEGA magnitude, error)
model = SingleStarModel(mist, **init_dict) # sets true values to those in init_dict
model.fit()
```

The likelihood of a live point is default calculated on a Gaussian curve, found in `guass_lnprob` in `likelihood.py`. Say the sensitivity of this filter (WFC3_HST_F336W) is 27, as in any magnitude greater than 27 is unreliable. We would then rather use a likelihood that is asymmetric, where live points with a value less than the cutoff sensitivity are treated normally and live points with a value greater than the cutoff sensitivity are assigned a likelihood equal to that of the cutoff point. This version of `likelihood.py` has a one-sided Gaussian likelihood function included for this purpose:

```
INCLUDE ONE-SIDED GAUSSIAN HERE PLACEHOLDER
```

It is also necessary to have somewhere to provide the set of these detection limits for specific filters. Because of the use of `numba` in `likelihood.py`, we instead create a dictionary called PLACEHOLDER at the top of `starmodel.py` to allow users to specify any detection limit for any filter used by isochrones. The other modifications to these two files are to implement these detection limits into the `star_lnlike` function, located in `likelihood.py` and called in `starmodel.py` when fitting. 

# ADDING BANDS TO BOLOMETRIC CORRECTION GRID

When creating a bolometric correction grid between different bands with `isochrones`, it is fairly straightforward to add any set of filters/bands from MIST (http://waps.cfa.harvard.edu/MIST/model_grids.html#bolometric). It can be done by modifying `../isochrones/mist/bc.py` and adding elements to the `phot_bands` dictionary. First, identify the table to add from MIST and copy the link. It should be of the form http://waps.cfa.harvard.edu/MIST/BC_tables/{NAME}.txz. Download the table and open one of the files to access the column names, which correspond to the filters/bands in the table. Add a list to the dictionary called `NAME` from the link and let the elements of this list be the names of the bands from the columns of the downloaded tables. Then, the first time one of these columns is called when creating a bolometric correction grid, the table will be downloaded to the `isochrones` package directory.
