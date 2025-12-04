# Season 1
## How to use github's search functionality to find code: MOM6 as an example.
Presenter: @dougiesquire (4/12).

Working example, [the MOM6 code repository](https://github.com/access-nri/mom6).
Contains all of the MOM6 Fortran code

Pro:

 - Easy and convenient
 - Allows you to see how parts of the model work

Cons:

 - Can only search on the default branch of the repo'.
 - GitHub doesn't actually index everything.
 - Can't search branches

Workflow suggestions: 

 - Initiate a seach by clicking in the search bar top-right or click `/`
 - Search for the word `"global"` for example, [using this](https://github.com/search?q=repo%3AACCESS-NRI%2FMOM6%20global&type=code).
 - Clicking on a line, will take you to specifically to that line, in that script.
 - Containing things in quotes will look for exact matches `"example phrase"`. Further [details here](https://docs.github.com/en/search-github/github-code-search/understanding-github-code-search-syntax#query-for-an-exact-match).
 - Regex searches are also possible.
 - You can also specify paths within the repo for example `path:src/core`.

Further details on search strategies are [here](https://docs.github.com/en/search-github/github-code-search/understanding-github-code-search-syntax).



## Distributed nature of MOM6 code across different development hubs / github repositories
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
