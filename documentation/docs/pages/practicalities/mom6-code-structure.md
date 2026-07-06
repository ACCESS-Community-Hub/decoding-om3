# Overview of MOM6 code structure

Presenter: Paul Spence (@PaulSpence)
Date: 26/03/2026.

Suggested resources in which this presentation was heavily based:

 - [Central MOM6 code](https://github.com/mom-ocean/MOM6);
 - [ACCESS-NRI MOM6 fork](https://github.com/acCESS-nri/mom6);
 - [Marshall Ward's talk on the structure of the MOM6 code base](https://www.marshallward.org/mom6workshop/internals.html#/mom6-directory-tree);
 - [Overview: MOM6 internals](https://www.youtube.com/watch?v=E8WKrESscc4).

Thanks to @marshallward for some great content!

Looking at the `mom6` [directory](https://github.com/aCCESS-NRI/mom6), we have:
 
 - `src/` -- model code and dynamical core solvers;
 - `config_src/` -- configurable components;
 - `pkg/` -- code directory but written by other people (eg. TEOS10 equation of state) linked into `src/`;
 - `doc/` -- documentation;
 - `ac/` -- autoconf build (build the model without mkf).

See 1 minute 20 in this [Overview: MOM6 internals](https://www.youtube.com/watch?v=E8WKrESscc4) for more details.

`Config_src` is particurly important. It has functions that call other model components (e.g NUOPC coupler):

 - `config_src/drivers`;
 - `solo_driver/` ocean-only (example [here](https://github.com/ACCESS-NRI/MOM6/tree/2026.01/config_src/drivers/solo_driver));
 - [NUOPC](https://github.com/ACCESS-NRI/MOM6/tree/2026.01/config_src/drivers/nuopc_cap) used in OM3 is an example of this.
 - other drivers (e.g. `ice_solo_driver` and `FMS_cap`)

Other configs are:

 - `config_src/memory` (e.g. [symmetric](https://github.com/ACCESS-NRI/MOM6/tree/2026.01/config_src/memory/dynamic_symmetric), [non-symmetric](https://github.com/ACCESS-NRI/MOM6/tree/2026.01/config_src/memory/dynamic_nonsymmetric), static.)
 - `config/src_infra` FMS1, FMS2
 - `config_src/external` BGC, data assimilation, python interface, etc.

Note: `config_src/external` just contains a set of dummy interfaces for external codebases that could be used with MOM6. For example there is no actual BGC code in as that lives in a different repository ([here](https://github.com/ACCESS-NRI/GFDL-generic-tracers)).

The `src` folder has model code and has directories:

 - `core/` -- main solvers such as initialisation and time-stepping 
 - `parameterizations` -- viscosity, mixing and diabatic
 - `tracer` -- tracer dynamics 
 - `ALE` -- vertical remapping
 - `diagnostics` -- diagnostic management
 - `user` -- preset focing and topography

Also see `framework/`, `equation_of_state/` etc

We then watched a little of this video ([Overview: MOM6 internals](https://www.youtube.com/watch?v=E8WKrESscc4)) focusing on modules, starting at 10 minutes.

Module guidelines:

 - "1 file per module rule".
 - explicit about dependencies (improve readability).
 - explicit about which functions are exposed publicly.
 - has object-like programming structures.
 - we don't put variables in modules. This is to ensure that we don't have global variables.
 - this rightward facing arrow `!>` shows what is being documents. (You'll notice there are also `!<`.).
 - Public parts of the interface are the ones that you can call elsewhere (as opposed to private ones that you can't). [Here an example](https://github.com/mom-ocean/MOM6/blob/08529ba87ea4ee6403446afc0c8b14f744f79c58/src/parameterizations/vertical/MOM_geothermal.F90#L25) of public subroutines.
 
 Here's some examples:

 - [MOM_diagnostics](https://github.com/mom-ocean/MOM6/blob/main/src/diagnostics/MOM_diagnostics.F90);
 - [MOM_diagnose_MLD](https://github.com/mom-ocean/MOM6/blob/main/src/diagnostics/MOM_diagnose_MLD.F90);
 - A simple example is the geothermal module ([here](https://github.com/mom-ocean/MOM6/blob/08529ba87ea4ee6403446afc0c8b14f744f79c58/src/parameterizations/vertical/MOM_geothermal.F90)).
