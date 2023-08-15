# FITTING WITH BAND DETECTION LIMITS

The versions of `likelihood.py` and `starmodel.py` here are meant to replace those in the isochrones package with minimal changes, so that normal use is not changed. The purpose of these changes is to allow the user to account for the sensitivity of filters when passing band magnitudes as initial values in a `StarModel` for pymultinest fitting. For example, 

```
from isochrones import mist, SingleStarModel
init_dict = {WFC3_HST_F336W_mag: (29, 2)} # (VEGA magnitude, error)
model = SingleStarModel(mist, **init_dict) # sets true values to those in init_dict
model.fit()
```

The likelihood of a live point is default calculated on a Gaussian curve, found in `guass_lnprob` in `likelihood.py`. Say the sensitivity of this filter (WFC3_HST_F336W) is 27, i.e. measurements above 27 would be unlikely to be detected. We would then rather use a likelihood that is asymmetric, where live points with a value less than the cutoff sensitivity are treated normally and live points with a value greater than the cutoff sensitivity are assigned a likelihood equal to that of the cutoff point. This version of `likelihood.py` has a one-sided Gaussian likelihood function included for this purpose:

```
def gauss_lnprob(val, unc, model_val):
    resid = val - model_val
    return LOG_ONE_OVER_ROOT_2PI + log(unc) - 0.5 * resid * resid / (unc * unc)

def one_sided_gauss_lnprob(limit, unc, model_val):
    resid = limit - model_val
    if resid >= 0:
        return gauss_lnprob(limit, unc, model_val)
    else:
        return gauss_lnprob(model_val, unc, model_val)```

The user can pass detection limits per filter when creating the `StarModel` instead of an initial value for magnitude. If the initial value is instead passed as

```
init_dict = {WFC3_HST_F336W_mag: (29, 2, False)} # (upper limit, error, measurement exists)
```

then 29 will be treated as the upper limit with an uncertainty of 2, with the `False` representing this is not an existing measurement. 

# ADDING BANDS TO BOLOMETRIC CORRECTION GRID

When creating a bolometric correction grid between different bands with `isochrones`, it is fairly straightforward to add any set of filters/bands from MIST (http://waps.cfa.harvard.edu/MIST/model_grids.html#bolometric). It can be done by modifying `../isochrones/mist/bc.py` and adding elements to the `phot_bands` dictionary. First, identify the table to add from MIST and copy the link. It should be of the form http://waps.cfa.harvard.edu/MIST/BC_tables/{NAME}.txz. Download the table and open one of the files to access the column names, which correspond to the filters/bands in the table. Add a list to the dictionary called `NAME` from the link and let the elements of this list be the names of the bands from the columns of the downloaded tables. Then, the first time one of these columns is called when creating a bolometric correction grid, the table will be downloaded to the `isochrones` package directory.
