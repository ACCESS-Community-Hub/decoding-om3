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

## How to find code that corresponds to a particular executable (both ACCESS-NRI and other MOM6 executables e.g. from Angus)
## How to find what diagnostics are available
## Overview of MOM6 configuration (input files etc)
## Searching through the MOM parameter docs and other output, e.g. what’s in what file, how to interpret maxCFL, truncations, warnings, errors
## Overview of MOM6 code structure - 
## How to contribute code back to MOM6
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
