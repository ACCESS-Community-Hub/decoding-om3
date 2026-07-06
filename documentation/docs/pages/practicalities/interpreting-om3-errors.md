# Interpreting OM3 Errors, Warnings, and Truncations

Presenter: @helenmacdonald 
Date: 16/04/2026

## Part 1: How do I know I have an error?

When your simulation is running, your control directory will have links to an archive and work directory along with a `access-om3.out` and `access-om3.err` file:
```
archive -> /scratch/project/user/access-om3/archive/my-buggy-run
work -> /scratch/project/user/access-om3/work/my-buggy-run
access-om3.out
access-om3.err
```
If your simulation has finished (you can check using `qstat`), payu will remove the work directory (after copying files into the archive directory) and move access-om3.out and access-om3.err into the archive directory. If your simulation has finished and you can still see some or all of these files/links in the control directory, or the expected output is missing from the archive directory it is likely that an error has occurred.

There are a few places to check for error messages, starting with these output files
```
access-om3.out
access-om3.err
1deg_jra55do_ia.e158765798
1deg_jra55do_ia.o158765798
```
Note that the last 2 of these files will have different names. They are usually some of the last files to be created and are of the form: `jobname.ejobid` and `jobname.ojobid`.
The error messagees are usually near the bottom of the files but can be a bit cryptic so also scroll up to see if there are some warnings about, e.g., a missing file.

It can also be helpful to look at log files in your work directory:
```
work/logfile.*.out
work/log/*
work/warnfile.000000.out
```
There are lots of files of the form `work/logfile.*.out`. Don’t look at them all, just pick one.
We can’t go through every error message that could arise but you should first try to classify your error:
 
- is it repeatable? (does it happen again if you do payu sweep; payu run?) 
- if not, it's likely a transient error, eg due to a brief hardware failure on Gadi. You can even [set up payu to automatically sweep and re-run](https://github.com/ACCESS-NRI/access-om2-configs/blob/release-01deg_jra55_iaf/config.yaml#L110-L112) if a particular error occurs.
- is it associated with a particular model component, or payu?
- does it happen in the initialisation, or after several timesteps?

A few more tips for debugging:

1. Look at whatever the last change was, often this is what caused the error.
2. Try putting the error message into google, or an AI agent
3. Search the [hive forum](https://forum.access-hive.org.au)
4. Check [Gadi status](https://opus.nci.org.au/spaces/Help/pages/399802963/System+Maintenance+and+Notices) just in case the issue is external
5. Ask a friend or post in the hive forum
6. [Search through the code-base](https://decoding-access-om3.readthedocs.io/season1/#how-to-use-githubs-search-functionality-to-find-code-mom6-as-an-example) for the error message or for key words
7.	Go back to a last working copy and slowly implement your changes, checking for errors along the way 

If all of these fail, there is the option to [use a debugger](https://docs.access-hive.org.au/models/build_a_model/build_source_code/#setting-up-the-debugger)

Note: If you struggled to find an answer, other people will as well so please consider posting the issue and solution to the [hive forum](https://forum.access-hive.org.au)

## Part 2: Truncation errors

See [here](https://access-om3-configs.access-hive.org.au/latest/Tips-and-tricks/) for more detiled notes on understanding truncation errors.

A common error is when the model goes numerically unstable, creating large velocities. MOM6 deals with these by truncating them (artificially setting them back to a realistic number). When MOM6 does this too many times it will end the simulation early with this error message:
```
Ocean velocity has been truncated too many times
```

The following parameters in MOM_input control this process.

```
U_TRUNC_FILE = "U_velocity_truncations" ! default = ""
                                ! The absolute path to a file into which the accelerations leading to zonal
                                ! velocity truncations are written. Undefine this for efficiency if this
                                ! diagnostic is not needed.
V_TRUNC_FILE = "V_velocity_truncations" ! default = ""
                                ! The absolute path to a file into which the accelerations leading to meridional
                                ! velocity truncations are written. Undefine this for efficiency if this
                                ! diagnostic is not needed.

MAXVEL = 6.0                    !   [m s-1] default = 3.0E+08
                                ! The maximum velocity allowed before the velocity components are truncated.
!CFL_TRUNCATE_RAMP_TIME = 7200.0 !   [s] default = 0.0
                                ! The time over which the CFL truncation value is ramped up at the beginning of
                                ! the run.
MAXTRUNC = 100000               !   [truncations save_interval-1] default = 0
                                ! The run will be stopped, and the day set to a very large value if the velocity
                                ! is truncated more than MAXTRUNC times between energy saves.  Set MAXTRUNC to 0
                                ! to stop if there is any truncation of velocities.
CFL_BASED_TRUNCATIONS = True    !   [Boolean] default = True
                                ! If true, base truncations on the CFL number, and not an absolute speed.
CFL_TRUNCATE = 0.5              !   [nondim] default = 0.5
                                ! The value of the CFL number that will cause velocity components to be
                                ! truncated; instability can occur past 0.5.
CFL_REPORT = 0.5                !   [nondim] default = 0.5
                                ! The value of the CFL number that causes accelerations to be reported; the
                                ! default is CFL_TRUNCATE.
CFL_TRUNCATE_RAMP_TIME = 7200.0 !   [s] default = 0.0
                                ! The time over which the CFL truncation value is ramped up at the beginning of
                                ! the run.
CFL_TRUNCATE_START = 0.0        !   [nondim] default = 0.0

```
Note that increasing `MAXTRUNC` usually doesn’t make the problem go away. In particular, when an issue comes up, it affects other variables such as sea surface elevation which can crash the simulation before you reach `MAXTRUNC`. As such it is worth looking for truncation errors if you are getting these sorts of errors:

```
WARNING from PE  1170: Extreme surface sfc_state detected
WARNING from PE   879: btstep: eta has dropped below bathyT
```

If your velocity is being truncated (regardless of if you reach `MAXTRUNC`) then you should also have one or both of these files (or similar):
```
work/V_velocity_truncations
work/U_velocity_truncations
```

These truncation files contain information on where and when the truncations occur and there is detailed information on how to interpret the files [here](https://access-om3-configs.access-hive.org.au/latest/Tips-and-tricks/#2-interpret-truncation-log-files).

There is a notebook to help investigate the location and timing of the truncation errors [here](https://github.com/ACCESS-NRI/access-eval-recipes/blob/main/ocean/Examine_truncation_data.ipynb).

Common ways to fix the issue are to:

1. decrease the timestep and
2. modify the bathymetry to remove “lumps” and “bumps” in the coastline and bathymetry that might be contributing to the instability. 

For 1: you can decrease the timestep in `MOM_input` by reducing parameters `DT` and `DT_THERM`. Changing timestep in ACCESS-OM3 is more involved due to the coupling - see [here](https://access-om3-configs.access-hive.org.au/latest/configurations/Overview/#timesteps). Make sure your coupling timestep (set in `nuopc.runseq`) is divisable by `DT` and `DT_THERM`.

For 2: bathymetry modification has a few potential pitfals, including the need to recreate other files (e.g. the mesh files).  There are some tools to help with bathymetry modification [here](https://github.com/COSIMA/bathymetry-tools). ACCESS-OM3 users are advised to start with a clone of [`make_om3_topo`](https://github.com/ACCESS-NRI/make_om3_topo) and check out the commit that created the `topog.nc` files used in their configuration. The commit is in the metadata of `topog.nc` - 'ncdump -h /path/to/file/topog.nc` (where `/path/to/file/` is documented in `config.yaml`) will get you this information. The `make_om3_topo` script goes through the steps needed to create the topography, including bathymetry modifications already performed. This workflow will also document any new changes to the bathymetry if you need to revisit this at a later date.

There is not a good universal rule to identify the "lumps and bumps". Sometimes it is obvious but sometimes it isn't and it can be helpful to ask some friends for their opinions.
