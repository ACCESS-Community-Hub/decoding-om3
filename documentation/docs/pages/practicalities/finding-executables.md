# How to find code that corresponds to a particular executable

## ACCESS-NRI executables
Presenter: @jbisits.

Scope: where and how to find which model components and source code are used in an OM3 configuration that are built with Spack (this applies to all ACCESS-NRI models).

### Through ACCESS-NRI release database (friendly)

The easiest way to find a related version is to use the [release database](https://reporting.access-nri-store.cloud.edu.au/release-provenance/releases
). For example, suppose we are interested in model version `2025.08.001` (listed in a `config.yaml` file in an `access-om3-configs` branch or directory if you have already cloned an experiment).

We can find this model version [here](https://reporting.access-nri-store.cloud.edu.au/release-provenance/releases/32/). Then clicking on any of the github icons for the relevant component takes you to the version of the code that was used in that release.

### Directly through GitHub (more complicated)

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
 - https://access-om3-configs.access-hive.org.au/latest/infrastructure/Building/
 - https://docs.access-hive.org.au/getting_started/spack/
 - https://docs.access-hive.org.au/models/build_a_model/build_source_code/
 - https://docs.access-hive.org.au/models/build_a_model/create_a_prerelease/ (not shown in this tutorial but can be very simple)

## COSIMA executables (e.g. `mom6-panan`) 
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
