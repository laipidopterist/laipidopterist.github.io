# Argo
These are some personal notes for working with and reading [Argo](https://argo.ucsd.edu/) data.

Most of this comes from the [Argo Online School](https://euroargodev.github.io/argoonlineschool/intro.html) and the [Argo data sources page](https://argo.ucsd.edu/data/).


# Getting the Data
There are a few different ways to download the data, which lives on an FTP server as well as an S3 bucket, and an overview
can be found at the Argo data, however, unless the point is live forecasting where you need live data, the easiest way seems
to be grabbing a monthly snapshot from [SEANOA](https://www.seanoe.org/).

Here's the link for the [2025-07-10 snapshot](https://www.seanoe.org/data/00311/42182/).

# Directory Structure
The directory structure of the downloaded file looks like this:

```
- aux
- doc
- geo
- dac
```

## Aux
`aux` contains auxiliary data (instrument metadata, cal info, quality control flags, etc).


## Doc
`doc` contains a bunch of documents written by the Argo organization that include cookbooks and technical recommendations.
One that I thought was particularly interesting for the (eventual) goal of replicating Sauzède et al. was "Processing
BGC-Argo particle scattering at the DAC level".


## Geo
`geo` contains gridded daily data, and is averaged and processed.


## Dac
`dac` (stands for Data Assembly Centers) contains the individual float data. Each DAC is responsible for its own fleet of
Argo floats, organized by a 7 digit number, the WMO number, assigned to each float upon deployment.[^1]


If you unzip and expand enough times, you'll end up with a structure like this:

```
- dac
  - coriolis_core
    - coriolis 
      - 1234567 (example WMO float number)
        - profiles
          - D1234567_NNN.nc
          - ...
          - ...
        - 1234567_meta.nc
        - 1234567_prof.nc
        - 1234567_Rtraj.nc
        - 1234567_tech.nc
```

The `D` prefix means "delayed-mode" (clean data, mostly). It might also be `R` for real-time, and there could be a `D`
suffix if the float takes measurements while descending, as well as ascending.

These are [NetCDF](https://www.unidata.ucar.edu/software/netcdf/) files, and there's a python library called [netCDF4](https://unidata.github.io/netcdf4-python/) to read them, though xarray also can read them.

```
import xarray as xr
import matplotlib.pyplot as plt
ds = xr.read_dataset("dac/coriolis_core/coriolis/1234567/profiles/D1234567_NNN.nc")

fig, ax = plt.subplots()

ax.plot(ds.TEMP[0], -ds.PRES[0])
ax.set_xlabel(ds.TEMP.long_name)
ax.set_xlabel(ds.PRES.long_name)
plt.show()
```

The units and range of the profile are stored in the data array:

```
>>> ds.TEMP.attrs
{'long_name': 'Sea temperature in-situ ITS-90 scale', 'standard_name': 'sea_water_temperature', 'units': 'degree_Celsius', 'valid_min': np.float32(-2.5), 'valid_max': np.float32(40.0), 'C_format': '%9.3f', 'FORTRAN_format': 'F9.3', 'resolution': np.float32(0.001)}
```

The pressure is in decibars, for what I'm assuming is reasonable oceanographic reasons[^2]:


```
>>> ds.PRES.attrs
{'long_name': 'Sea water pressure, equals 0 at sea-level', 'standard_name': 'sea_water_pressure', 'units': 'decibar', 'valid_min': np.float32(0.0), 'valid_max': np.float32(12000.0), 'C_format': '%7.1f', 'FORTRAN_format': 'F7.1', 'resolution': np.float32(1.0), 'axis': 'Z'}
```

[^1]: There also seems to be _five_ digit numbers in the dataset; I'm not sure if this is group information (i.e., 12345 contains
information about the floats in the range 1234500 - 1234599) or if it's an older serial number.

[^2]: I've since learned that an increase in depth of 1m is roughly equal to an increase of 1 decibar, so it's convenient.
