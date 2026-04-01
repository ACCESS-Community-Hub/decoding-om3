# Season 1

## General info on decoding MOM6 sessions
- The early sessions of these “Decoding MOM6” segments will talk through the tools that might be useful to navigate the MOM6 code
- The later sessions will dive deeper into the code itself

## How to use github's search functionality to find code: MOM6 as an example.

Presenter: @dougiesquire

Searching [the MOM6 code repository in the ACCESS-NRI GitHub org](https://github.com/access-nri/mom6).
Contains all of the MOM6 Fortran code used in ACCESS-OM3

### Quick summary of GitHub search functionality

Pros:

 - Easy and convenient
 - Allows you to see how parts of the model work

Cons:

 - Can only search on the default branch of the repo.
 - GitHub doesn't actually index everything.


### More detailed notes

- Why use the GitHub search bar?
    - See how a specific piece of code is written - how is it implemented in MOM6? Need to search through the code
- Dougie’s screen: Looking at [MOM6 fork on ACCESS-NRI GitHub org](https://github.com/ACCESS-NRI/MOM6)
    - Can click on search at top or click `/` to start a search
    - Eg [search `global`](https://github.com/search?q=repo%3AACCESS-NRI%2FMOM6%20global&type=code)
        - Will return all files in this repo with “Global” (not case sensitive by default)
        - Get the summary of the files, and can click to expand more lines of code
        - Can click on the line itself and it will take you to that line in the file
    - Can search on multiple terms
        - Eg `Global Mean` - searches all files with global or mean
        - Eg `“Global Mean”` - search for specific phrase
        - Eg `“Global mean ocean salinity”` - can search directly for this diagnostic
            - Finds where it’s registered, and can backtrack to find the code
    - Can search on patterns (regex) rather than words
        - Eg search `Global` but only at specific path: `Global path:src/core`
    - Fast, powerful, easy to use
    - Can share searches with people (can’t do this if you are, eg, using “grep” on Gadi terminal)
- Gotchas
    - Can only search on default branch
    - GitHub doesn’t index everything, and opaque which code is indexed
        - Eg doesn’t index vendor code, but not clear what that means
        - If you search something and it seems weird that nothing is found, then this might be the main issue

#### Questions from the audience
- _Q: Can it organize the search in order of the subroutines called?_
    - MOM6 docs does include schematics on subroutines that are called from specific file
        - But may not be generated anymore, so may be out of date
- _Q: Can searches be case sensitive?_
    - Yes, you can use regex: `/(?-i)Global/`
    - By default they aren’t case sensitive
- _Q: What is regex?_
    - Language for pattern matching
    - Stands for regular expression
    - Lots of resources online
        - Just be sure to use the correct version of regex, as there are different types
    - Can click “Search syntax tips” - link at bottom of search popup in GitHub - provides quick help for lots of common regex uses
- _Q: OM3 configurations are all on a branch - does that mean we’ve eliminated the ability to search through OM3 configurations?_
    - Yes, we can’t search OM3 configurations at the moment using this tool, because there is almost nothing on the default branch. This is definitely a limitation.
    - Some parameter documentation has been moved to main branch specifically to be able to search on them
    - There are other ways to search through other branches, but no examples were given
    - Lots of people would like GitHub to add functionality to search on a branch other than the default, but not the case currently
 
### More resources
[Understanding GitHub Code Search syntax](https://docs.github.com/en/search-github/github-code-search/understanding-github-code-search-syntax#query-for-an-exact-match)


## Lessons learned from MOM6 code development

Presenter: @jbisits (11/12/2025).
!!! tip
    Take-home message: when starting development, take care with which version of the MOM6 code repository you fork from.

### Background

When @jbisits started work on MOM6, he went to google and found the MOM6 codebase and then made a fork of `mom-ocean/MOM6` (the [central consortium repository](https://github.com/mom-ocean/MOM6)). Actually if one is looking to contribute to the MOM6 codebase _from_ Australia, there is an [access-nri/MOM6](https://github.com/access-nri/mom6) fork. They are different forks with different branches and code bases! 

### How do I find out which version of MOM6 I am currently using?

Go to the OM3 configuration that you are using [example](https://github.com/ACCESS-NRI/access-om3-configs/blob/release-MC_25km_jra_iaf/config.yaml
) and find the `config.yaml` following lines:

Note these lines:
```yaml
modules:
    use:
        - /g/data/vk83/modules
    load:
        - access-om3/2025.08.001
```

Specifically note that this line `access-om3/2025.08.001` highlights the [Spack bundle package](https://github.com/ACCESS-NRI/access-spack-packages/blob/main/packages/access-om3/package.py) and git tag that is being used for MOM6 code. One can then match this tag name `2025.08.001` from the model deployment repository, this link [lists all the tags](https://github.com/ACCESS-NRI/ACCESS-OM3/tags), and here is the tag we are looking for: [`2025.08.001`](https://github.com/ACCESS-NRI/ACCESS-OM3/releases/tag/2025.08.001) in this case.

Further information:

 - [MOM6 fork management for being a development node](https://github.com/ACCESS-NRI/MOM6/wiki);
 - [OM3 build system and deployment](https://access-om3-configs.access-hive.org.au/infrastructure/Building/);
 - [NOAA-GFDL MOM6 development guide](https://github.com/NOAA-GFDL/MOM6-examples/wiki/Developers-guide);
 - [MOM6 development presentation](https://www.marshallward.org/mom6workshop/develop.html) by @marshallward.

## How to find code that corresponds to a particular executable
### ACCESS-NRI executables
Presenter: @jbisits.

Scope: where and how to find which model components and source code are used in an OM3 configuration that are built with Spack (this applies to all ACCESS-NRI models).

#### Through ACCESS-NRI release database (friendly)

The easiest way to find a related version is to use the [release database](https://reporting.access-nri-store.cloud.edu.au/release-provenance/releases
). For example, suppose we are interested in model version `2025.08.001` (listed in a `config.yaml` file in an `access-om3-configs` branch or directory if you have already cloned an experiment).

We can find this model version [here](https://reporting.access-nri-store.cloud.edu.au/release-provenance/releases/32/). Then clicking on any of the github icons for the relevant component takes you to the version of the code that was used in that release.

#### Directly through GitHub (more complicated)

For example looking at `ACCESS-OM3`, we start by looking at the [configuration repository](https://github.com/access-nri/access-om3-configs/): `ACCESS-NRI/access-om3-configs`. Note that the main branch does not contain configurations -- you have to look at the branches.

You may wish to know what are the model components being used in [a particular release](https://github.com/ACCESS-NRI/access-om3-configs/tags)? Looking at the release: `release-MC_25km_jra_iaf-1.0-beta` we can browse the repository at the relevant [tag here](https://github.com/ACCESS-NRI/access-om3-configs/tree/release-MC_25km_jra_iaf-1.0-beta).

Then in the `config.yaml`, see [the model software version](https://github.com/ACCESS-NRI/access-om3-configs/blob/95ac4456b2e08a3dc1e6476e8dc96900005151ce/config.yaml#L18-L23):

```yaml
modules:
    use:
        - /g/data/vk83/modules
    load:
        - access-om3/2025.08.001
```

`access-om3/2025.08.001` refers to [this repository](https://github.com/access-nri/access-om3) with this at the tag: `2025.08.001`. To find the tags one goes to [the repository](https://github.com/ACCESS-NRI/ACCESS-OM3) --> Releases --> Tags. Here is the related `access-om3` release:
https://github.com/ACCESS-NRI/ACCESS-OM3/releases/tag/2025.08.001

Browsing the related `spack.yaml` we can see the versions of the components in the build, [for example](https://github.com/ACCESS-NRI/ACCESS-OM3/blob/7439c61acbe167abd79fc9be4dc333fadb9ac0cb/spack.yaml#L10-L82) for CICE and MOM6 we have:
```yaml
    access-cice:
      require:
        - '@CICE6.6.1-0'
        - io_type=PIO
        - 'fflags="-march=sapphirerapids -mtune=sapphirerapids -unroll"'
        - 'cflags="-march=sapphirerapids -mtune=sapphirerapids -unroll"'
        - 'cxxflags="-march=sapphirerapids -mtune=sapphirerapids -unroll"'
    access-mom6:
      require:
        - '@2025.07.000'
        - 'fflags="-march=sapphirerapids -mtune=sapphirerapids -unroll"'
        - 'cflags="-march=sapphirerapids -mtune=sapphirerapids -unroll"'
        - 'cxxflags="-march=sapphirerapids -mtune=sapphirerapids -unroll"'
```

Like before, we go to the related `ACCESS-NRI` repositories and find the related tags, in this instance they are:
 - [access-CICE](https://github.com/ACCESS-NRI/CICE/releases/tag/CICE6.6.1-0);
 - [access-mom6](https://github.com/ACCESS-NRI/MOM6/releases/tag/2025.07.000).

Note that the above two repositories are forks of the upstream repositories ([CICE](https://github.com/ESCOMP/CICE) and [MOM6](https://github.com/mom-ocean/MOM6))

For more information and other related steps: 
 - https://access-om3-configs.access-hive.org.au/infrastructure/Building/
 - https://docs.access-hive.org.au/getting_started/spack/
 - https://docs.access-hive.org.au/models/build_a_model/build_source_code/
 - https://docs.access-hive.org.au/models/build_a_model/create_a_prerelease/ (not shown in this tutorial but can be very simple)

----

### COSIMA executables (e.g. `mom6-panan`) 
Presenter: @angus-g.

Scope: where and how to find which model components and source code are used in a COSIMA configuration built with ninja.

COSIMA made executables use a different build system and way of tracking provenance. Here's an example using the `mom6-panan`. Going to [the relevant line](https://github.com/COSIMA/mom6-panan/blob/master/config.yaml#L17) in the `config.yaml` we have: `exe: /g/data/ik11/inputs/mom6/bin/symmetric_FMS2-e7d09b7`

If one then goes on Gadi to the related folder, one finds:
`/g/data/ik11/inputs/mom6/bin/manifest.yaml`

Looking inside the manifest file we see a series of executables with information related to the major components (MOM6, SIS2, FMS) with the git hash used. The compiler flags are also included but not the more minor components. Below is an example:
```yaml
exe: "symmetric_FMS2-e7d09b7"
  build-date: 2022-01-20
  sha256: "e7d09b7687fba4cf0693adf3c1a5f653dee312df996b99da384c64c279a53eef"
  git:
    - component: MOM6
      ref: "93968bd8ec1a993c705846ec91c4cf33b86bc4fb"
    - component: SIS2
      ref: "d18605eacdd2c92b8262458fcad107c546dd080f"
    - component: FMS
      ref: "2d373d20af28b9d8e2cc0d0acf2334fcff457e47"
  modules:
    - "intel-mkl/2020.2.254"
    - "python3/3.8.5"
    - "netcdf/4.7.4p"
    - "openmpi/4.1.2"
    - "intel-compiler/2021.4.0"
  keywords: [SIS2, symmetric, FMS2]
  flags:
    fflags: "-fno-alias -auto -safe-cray-ptr -ftz -assume byterecl -i4 -r8 -nowarn -sox -g -O2 -debug minimal -fp-model precise -qoverride-limits -xHost"
    cflags: "-D__IFC -sox -g"
```

With the above information, one could then compile using the `mom6-ninja-nci` workflow written by @angus-g -- [see here](https://github.com/angus-g/mom6-ninja-nci).

More information:
 - https://github.com/angus-g/mom6-ninja-nci
 - https://github.com/COSIMA/access-om2/wiki/Developers-guide (possibly out of date)
 - http://github.com/cosima/access-om3 (deprecated!)

## How to find what diagnostics are available in MOM6
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

### How to add a diagnostic
One can simply add lines to the diag_table file ([example](https://github.com/ACCESS-NRI/access-om3-configs/blob/release-MC_25km_jra_iaf/diagnostic_profiles/diag_table_standard)). This is _not_ the suggested workflow, we suggest editing this yaml file:
https://github.com/ACCESS-NRI/access-om3-configs/blob/dev-MC_25km_jra_iaf%2Bwombatlite/diag_table_source.yaml

This yaml file is used by `make_diag_table.py` (latest version: https://github.com/COSIMA/make_diag_table) to create a diag_table file specifying MOM diagnostics. 

Further information on the MOM diag_table format is here:

 - https://github.com/mom-ocean/MOM5/blob/master/src/shared/diag_manager/diag_table.F90
 - https://mom6.readthedocs.io/en/main/api/generated/pages/Diagnostics.html
 - https://www.youtube.com/watch?v=D_J8eg3G80o


### How to find how and where a diagnostic is calculated
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

## Overview of MOM6 configuration (input files etc)
Presenters: @aekiss @claireyung

Configuration and input files when running MOM6 "standalone" (FMS coupler). Actually, running MOM6 in an idealised way only requires 3 input files. Here is an example taken from Claire's `IS-PG-MOM6` repository:

 - `MOM_input` -- parameter settings (name defined in 'MOM_input').
 - `diag_table` -- diagnostics.
 - `input.nml` -- run settings (calendar, MOM_input names etc).

As this is an idealised test case, much of the configuration (forcing, geometry etc) are defined in `common/MOM_input`. MOM_input file typically contain only the non-default values that are needed. A full list of parameters  be found in the corresponding `MOM_parameter_doc.all` file ([example](https://github.com/claireyung/IS-PG-MOM6/blob/main/icemount-layer-LSPR/MOM_parameter_doc.all)) which is generated by the model at run-time. In Claire's case, she builds "perturbations" of the base case ("common") by modifying these files; in `icemount-layer-LSPR` ([here](https://github.com/claireyung/IS-PG-MOM6/tree/main/icemount-layer-LSPR)) you can see the symlinks to the `MOM_input`, `diag_table`, `input.nml` files discussed above. Except that now there is a MOM_override file ([here](https://github.com/claireyung/IS-PG-MOM6/blob/main/icemount-layer-LSPR/MOM_override)) which overrides any settings in `MOM_input`. 

MOM6 also has an in-built sea-ice model SIS2. An example configuratin is [here](https://github.com/cosima/mom6-panan/tree/master). You can see that there are more configuration files (additional to `MOM_input`, `diag_table`, `input.nml`): 

 - `field_table`;
 - `SIS_input` SIS2 input files;
 - `data_table` forcing files if using FMS coupler

ACCESS-NRI ACCESS-OM3 uses a different coupler (NUOPC) compared to the above two examples. It also uses a different "standalone" sea-ice model CICE. So whilst the MOM elements discussed above remain the same. Additional files are required for NUOPC to couple the components. 

![ACCESS OM3 diagram](https://access-om3-configs.access-hive.org.au/assets/nuopc_overview.png)
source: https://access-om3-configs.access-hive.org.au/infrastructure/Architecture/

There's a description of the files found in a configuration here: https://access-om3-configs.access-hive.org.au/configurations/Overview/

OM3 configurations are stored here:
https://github.com/acCESS-nri/access-om3-configs

Configurations that have a `{dev|release}-` prefix are the ones to focus on. Briefly, the configurations branches are named with the following
`{dev|release}-{MODEL_COMPONENTS}_{nominal_resolution}km_{forcing_data}_{forcing_method}[+{modifier}]`

Further details [here](https://access-om3-configs.access-hive.org.au/#access-om3-configs-overview).

Users are welcome to fork the access-nri configuration repository and share what changes/additions they make to configurations. ACCESS-NRI is also interested in helping support users share configurations ([process outlined here](https://access-om3-configs.access-hive.org.au/contributing/Add-Supported-Config/)). We are also keeping a list of key experiments used for [development here](https://access-om3-configs.access-hive.org.au/Experiments/).

Handy resources:

 - ACCESS-NRI's OM3 configurations [live here](https://github.com/acCESS-nri/access-om3-configs);
 - ACCESS-NRI [ACCESS-OM3 configuration files explanation](https://access-om3-configs.access-hive.org.au/configurations/Overview/); 
 - [MOM6 runtime parameters format (input.nml, MOM_input)](https://mom6.readthedocs.io/en/main/api/generated/pages/Runtime_Parameter_System.html);
 - [diag_table](https://mom6.readthedocs.io/en/main/api/generated/pages/Diagnostics.html) ;
 - [Another explanation of config files from MOM6 regional](https://regional-mom6.readthedocs.io/en/latest/mom6-file-structure-primer.html);
 - [AOS MOM6 tutorial 2022: Running and controlling MOM6](https://www.youtube.com/watch?v=94m3CMTwJ1E) (e.g. ~15 minutes)

## Overview of MOM6 code structure 
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

## Specifying model outputs via `diag_table` or `make_diag_table`
Presenter: @chrisb13 and @aekiss
Date: 02/04/2026

### Understanding the MOM6 diag_table.
Presenter: @chrisb13 (channeling Alistair Adcroft)

Resources:

 - [Tutorial: Running and controlling MOM6](https://www.youtube.com/watch?v=94m3CMTwJ1E) (31 to 37 minutes);
 - [MOM6 diagnostics on readthedocs](https://mom6.readthedocs.io/en/dev-gfdl/api/generated/pages/Diagnostics.html);
 - [ACCESS hive docs configuration MOM6 diagnostics](configuring-mom6-diagnostics);
 - [Dougie on adding diagnostics](https://decoding-access-om3.readthedocs.io/decoding_mom6/#how-to-add-a-diagnostic).

Check out [the video](https://www.youtube.com/watch?v=94m3CMTwJ1E) from Alistair Adcroft (30 minutes 57 sec). Briefly:

4 parts to the diag table:

 - Label (title section) -- required;
 - Date (title section) -- required -- reference date for realistic models is typically `1900` whereas `0 0 1` is often used in realistic setups;
 - File section -- this section defines an arbitrary number of files that will be created. Each file is limited to a single rate of either sampling or time-averaging;
 - Field section -- an arbitrary number of lines, one per diagnostic field.

In the **file** section, we have:

> "file_name",  output_freq,  "output_freq_units",  file_format,  "time_axis_units",  "time_axis_name"

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
 - reduction_method: “none” means sample or snapshot. “average” or “mean” performs a time-average. “min” or “max” diagnose the minium or maxium over each time period.
 - regional_section: “lon_min lon_max lat_min lat_max vert_min vert_max” limits the diagnostic to a region (“none” means global output).
 - packing: Data representation in the file. 1 means “real*8”, 2 means “real*4”, 4 mean 16-bit integers, 8 means 1-byte.


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
 - packing: 2 means “real*4”

More examples [here](https://mom6.readthedocs.io/en/dev-gfdl/api/generated/pages/Diagnostics.html#example).

Also, from earlier sessions recall that the list of available diagnostics is dependent on the particular configuration of the model. For this reason the model writes a record of the available diagnostic fields at run-time into a file “available_diags", [here's an example](https://github.com/ACCESS-NRI/access-om3-configs/blob/release-MC_25km_jra_iaf/docs/available_diags.000000) from ACCESS-OM3.

### Using the COSIMA created `make_diag_table` workflow.
Presenter: @aekiss

## OM3 runtime output files  (Chris)
Presenter: @chrisb13 
Date: 09/04/2026

## Interpreting OM3 maxCFL, truncations, warnings, errors (Helen)
Presenter: @ helenmacdonald 
Date: 16/04/2026

## How to contribute code back to MOM6 
Presenter: @ jbisits and @ dougiesquire 
Date: 23/04/2026

# Season 2
## Navier Stokes -> stacked shallow water (adiabatic)
## Generalised vertical coordinates
## Vertical Lagrangian remapping
## Pressure forces
## Coriolis term
## Pressure solver - barotropic / baroclinic split
## Timestepping / Advection schemes
## Vertical velocity diagnostics
## MEKE
## EPBL
## KPP
## Submesoscale parameterisation
## Ice shelf code
## Ocean fresh-water forcing / water balance
## How to create a new MOM6 diagnostic
## Horizontal viscosity
## Advection schemes
## Bottom drag (Callum? Luwei?)
## Opacity for shortwave penetration
## For low res: GM / Redi eddy params
## Explicit tides, self attraction and loading
## Regional boundary conditions
## Bulk formulae
## Fortran 101 - control structure, where parameters are defined, some keywords e.g. private, intent, submodule, use module etc
## How to work out how a diagnostic was calculated (general syntax + keywords to search for)
## Different initialisation options - thickness + tracers, topography (all these preset options)
## Coord config vs use_regridding (layer vs ale mode) - how to change your vertical coordinate or target coordinate and what it means
## Regridding of diagnostics and vanished layer output - how to change in diag_table, what it means for output
## Equations of state and different types of salinity, temperature in initialisation, model running and output
## What does the sea ice model do for the ocean ?
## MOM5/MOM6 code differences?
## Shear mixing
## Internal tide mixing
## Lee wave mixing
## Bottom boundary mixing
## Background mixing
