# How to find what diagnostics are available in MOM6

Presenter: @dougiesquire.

If one goes to to a run directory on Gadi, then you can look at `available_diags.000000`. This file tells you all the available diagnostics. For each of the disagnostics it lists, it tells you:

 - which diagnostics are `[used]` or ` [Unused]`;
 - the units;
 - description of the variable;
 - name of the variable;
 - which grid points the diagnostic is output on.

Here's an example:
```
"KHMEKE_v"  [Unused]
    ! modules: {ocean_model,ocean_model_d2}
    ! dimensions: xh, yq
    ! long_name: Meridional diffusivity of MEKE
    ! units: m2 s-1
    ! cell_methods: xh:mean yq:point
```

If you are using an access-om3-configs configuration, we keep up to date versions of that file alongside our configurations, [here](https://github.com/ACCESS-NRI/access-om3-configs/tree/release-MC_25km_jra_iaf/docs). So the `available_diags.000000` file is viewed [here](https://github.com/ACCESS-NRI/access-om3-configs/blob/release-MC_25km_jra_iaf/docs/available_diags.000000) for the `release-MC_25km_jra_iaf` configuration.

## How to add a diagnostic

One can simply add lines to the diag_table file ([example](https://github.com/ACCESS-NRI/access-om3-configs/blob/release-MC_25km_jra_iaf/diagnostic_profiles/diag_table_standard)). This is _not_ the suggested workflow, we suggest editing this yaml file:
https://github.com/ACCESS-NRI/access-om3-configs/blob/dev-MC_25km_jra_iaf%2Bwombatlite/diag_table_source.yaml

This yaml file is used by `make_diag_table.py` (latest version: https://github.com/COSIMA/make_diag_table) to create a diag_table file specifying MOM diagnostics. 

Further information on the MOM diag_table format is here:

 - https://github.com/mom-ocean/MOM5/blob/master/src/shared/diag_manager/diag_table.F90
 - https://mom6.readthedocs.io/en/main/api/generated/pages/Diagnostics.html
 - https://www.youtube.com/watch?v=D_J8eg3G80o


## How to find how and where a diagnostic is calculated

Suppose one wants to know how a field is calculated. Taking our example above (and making sure we are looking at the correct mom6 code base for our configuration) we can search the mom6 code base for the `long_name`, namely `Meridional diffusivity of MEKE`, i.e. [here](https://github.com/ACCESS-NRI/MOM6/blob/c664721ebd58c033964b502e7fcdcccd05f02947/src/parameterizations/lateral/MOM_MEKE.F90#L1706).

So the relevant block is
```fortran
    CS%id_KhMEKE_v = register_diag_field('ocean_model', 'KHMEKE_v', diag%axesCv1, Time, &
     'Meridional diffusivity of MEKE', 'm2 s-1', conversion=US%L_to_m**2*US%s_to_T)
```

Helps us find [the actual array that is being output](https://github.com/ACCESS-NRI/MOM6/blob/c664721ebd58c033964b502e7fcdcccd05f02947/src/parameterizations/lateral/MOM_MEKE.F90#L954-L955):
```fortran
  if (CS%id_KhMEKE_v>0) call post_data(CS%id_KhMEKE_v, Kh_v, CS%diag)
```
namely `Kh_v`.

Note that changing CICE outputs involves modifying the relevant namelist: `ice_in` ([here is an example](https://github.com/ACCESS-NRI/access-om3-configs/blob/649e9f4877c1d7ab8caaa8607be8ded96c823648/ice_in#L112)).
