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
