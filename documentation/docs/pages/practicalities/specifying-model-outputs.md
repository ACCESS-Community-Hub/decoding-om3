# Specifying model outputs via `diag_table` or `make_diag_table`

Presenter: @chrisb13 and @aekiss
Date: 02/04/2026

## Understanding the MOM6 diag_table

Presenter: @chrisb13 (Chris Bull -- channeling Alistair Adcroft)

Resources:

 - [Tutorial: Running and controlling MOM6](https://www.youtube.com/watch?v=94m3CMTwJ1E&t=1860s) (31 to 37 minutes);
 - [MOM6 diagnostics on readthedocs](https://mom6.readthedocs.io/en/dev-gfdl/api/generated/pages/Diagnostics.html);
 - [ACCESS hive docs configuration MOM6 diagnostics](https://docs.access-hive.org.au/models/run_a_model/run_access-om3/#configuring-mom6-diagnostics);

4 parts to the diag table:

 - Label (title section) -- required;
 - Date (title section) -- required -- reference date (year month day hour minute second) for realistic models is typically `1900 1 1 0 0 0` whereas `1 1 1 0 0 0` is often used in idealised setups;
 - File section -- this section defines an arbitrary number of files that will be created. Each file is limited to a single rate of either sampling or time-averaging;
 - Field section -- an arbitrary number of lines, one per diagnostic field.

In the **file** section, we have:

> "file_name",  output_freq,  "output_freq_units",  file_format,  "time_axis_units",  "time_axis_name"

plus optional extras (see [MOM6 docs](https://mom6.readthedocs.io/en/dev-gfdl/api/generated/pages/Diagnostics.html#file-section)).

Here's an example from Claire Yung `MOM6-examples-z/diag_table` ([link](https://github.com/claireyung/IS-PG-MOM6/blob/main/MOM6-examples-z/diag_table)):

> "GOLD Experiment"
> 1 1 1 0 0 0

Claire [also has](https://github.com/claireyung/IS-PG-MOM6/blob/3ba9863f52e075a3f588c34406d03f2b22c85fe8/MOM6-examples-z/diag_table#L7):

> "prog",     6,"hours",1,"days","Time"

Here's how to interpret this:

 - "file_name": "prog" (excludes the `.nc` extension)
 - output_freq: 6
 - "output_freq_units": "hours"
 - file_format: 1 (Always set to 1, meaning netcdf.)
 - "time_axis_units": "days" (units to use for the time-axis in the file. Valid values are “years”, “months”, “days”, “hours”, “minutes” or “seconds”)
 - "time_axis_name": "Time" (The name of the time-axis, usually “Time”)

In the **field** section, we have:

> "module_name",  "field_name",  "output_name",  "file_name",  "time_sampling",  "reduction_method",  "regional_section",  packing

 - module_name: Name of the component model. 
 - field_name: The name of the variable as registered in the model.
 - output_name: The name of the variable as it will appear in the file.
 - file_name: One of the files defined above in the section File section (a target).
 - time_sampling: Always set to “all”.
 - reduction_method: “none” means sample or snapshot. “average” or “mean” performs a time-average. “min” or “max” diagnose the minimum or maximum over each time period. [Other options](https://github.com/mom-ocean/MOM5/blob/6bdbdd4892543bbade921fa3224b2530d93c6f40/src/shared/diag_manager/diag_table.F90#L169-L182) are also available.
 - regional_section: “lon_min lon_max lat_min lat_max vert_min vert_max” limits the diagnostic to a region (“none” means global output).
 - packing: Data representation in the file. 1 means double precision (64 bit real), 2 means single precision (32 bit real), 4 means packed 16-bit integers, 8 means packed 1-byte.

https://github.com/claireyung/IS-PG-MOM6/blob/3ba9863f52e075a3f588c34406d03f2b22c85fe8/MOM6-examples-z/diag_table#L22-L33

Picking up on Claire's example from earlier, [we have several fields](https://github.com/claireyung/IS-PG-MOM6/blob/3ba9863f52e075a3f588c34406d03f2b22c85fe8/MOM6-examples-z/diag_table#L22-L33) that will end up in the `prog.nc` file ([defined here](https://github.com/claireyung/IS-PG-MOM6/blob/3ba9863f52e075a3f588c34406d03f2b22c85fe8/MOM6-examples-z/diag_table#L7)), namely:

> "ocean_model","u","u","prog","all",.false.,"none",1
 
> "ocean_model","v","v","prog","all",.false.,"none",1
 
> "ocean_model","h","h","prog","all",.false.,"none",1
 
> "ocean_model","temp","temp","prog","all",.false.,"none",2

> "ocean_model","salt","salt","prog","all",.false.,"none",2

So taking the first one as an example:

 - module_name: Name of the component model. 
 - field_name: we will output the variable `u`.
 - output_name: when we output `u`, we'll call it `u`.
 - file_name: target file is "prog".
 - time_sampling: here it is “all” but this could be "mean" (note that you cannot mix and match within a file, nor can you have different frequencies).
 - reduction_method: `.false.` means no time reduction.
 - regional_section: “none” means no limited region.
 - packing: 2 means “real*4” (single precision)

More examples [here](https://mom6.readthedocs.io/en/dev-gfdl/api/generated/pages/Diagnostics.html#example).

Also, from earlier sessions recall that the list of available diagnostics is dependent on the particular configuration of the model. For this reason the model writes a record of the available diagnostic fields at run-time into a file “available_diags", [here's an example](https://github.com/ACCESS-NRI/access-om3-configs/blob/release-MC_25km_jra_iaf/docs/available_diags.000000) from ACCESS-OM3.

## Using the COSIMA created `make_diag_table` workflow

Presenter: @aekiss (Andrew Kiss)

[`make_diag_table`](https://github.com/COSIMA/make_diag_table) is a script that generates a `diag_table` file from a YAML file `diag_table_source.yaml`.
It can be run with
```
python /g/data/vk83/apps/make_diag_table/make_diag_table.py
```
which reads `diag_table_source.yaml` and writes `diag_table`, overwriting it if it already exists.
`diag_table_source.yaml` covers every feature of `diag_table`, so when using `make_diag_table` the normal practice is to only edit `diag_table_source.yaml`.

Why use `make_diag_table`? In ACCESS-OM2 and ACCESS-OM3 we often want to save one file per variable, using an informative and standardised filename convention, e.g.
```
ocean-2d-surface_salt-1-daily-mean-ym_1958_01.nc
ocean-2d-surface_salt-1-monthly-mean-ym_1958_01.nc
ocean-2d-surface_temp-1-daily-mean-ym_1958_01.nc
ocean-2d-surface_temp-1-monthly-mean-ym_1958_01.nc
ocean-2d-surface_temp-1-monthly-min-ym_1958_01.nc
ocean-2d-swflx-1-monthly-mean-ym_1958_01.nc
ocean-2d-tau_x-1-monthly-mean-ym_1958_01.nc
ocean-2d-tau_y-1-monthly-mean-ym_1958_01.nc
ocean-2d-temp_int_rhodz-1-monthly-mean-ym_1958_01.nc
ocean-2d-temp_xflux_adv_int_z-1-monthly-mean-ym_1958_01.nc
ocean-2d-temp_yflux_adv_int_z-1-monthly-mean-ym_1958_01.nc
ocean-2d-tx_trans_int_z-1-monthly-mean-ym_1958_01.nc
ocean-2d-wfiform-1-monthly-mean-ym_1958_01.nc
ocean-2d-wfimelt-1-monthly-mean-ym_1958_01.nc
ocean-3d-age_global-1-monthly-mean-ym_1958_01.nc
ocean-3d-buoyfreq2_wt-1-monthly-mean-ym_1958_01.nc
ocean-3d-diff_cbt_t-1-monthly-mean-ym_1958_01.nc
ocean-3d-dzt-1-monthly-mean-ym_1958_01.nc
ocean-3d-pot_rho_0-1-monthly-mean-ym_1958_01.nc
ocean-3d-pot_rho_2-1-monthly-mean-ym_1958_01.nc
ocean-3d-pot_temp-1-monthly-mean-ym_1958_01.nc
ocean-3d-salt-1-monthly-mean-ym_1958_01.nc
ocean-3d-temp-1-monthly-mean-ym_1958_01.nc
```
This would involve a lot of repetitious and error-prone fiddling around if done by hand within `diag_table`. `make_diag_table` solves this problem by allowing you to specify a standardised file name convention and automatically generate file names for each variable you save.

Here's [an example `diag_table_source.yaml`](https://github.com/COSIMA/make_diag_table/blob/master/diag_table_source.yaml). It is thoroughly commented and should be fairly intelligible.

It is in two sections:

- `global_defaults` which sets the defaults used for every file and field unless overridden in `defaults` in the `diag_table` section
  - the `file_name` list the components which are concatenated to form a standardised filename; their values are defined below
- `diag_table` which defines the diagnostics to appear in the generated `diag_table`
  - These are grouped together in categories, which are variables that have a common set of parameters (such as `reduction_method` or `output_freq`) defined in `defaults` (which override `global_defaults`)
    - within each category, `fields` is a dictionary containing all the variables in that category.

**So to add an output field to an existing category, all you need to do is add its name to the `fields` dictionary and run**
```
python /g/data/vk83/apps/make_diag_table/make_diag_table.py
```
This will update `diag_table` to have new file and field entries, with a standardised file name.
