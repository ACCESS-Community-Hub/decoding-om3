# Shortwave penetration in MOM6

## The general stuff

Incoming shortwave radiation (SW) penetrates and heats a finite layer of the water column. Getting the vertical penetration right is important for SST and mixed-layer dynamics.

Absorption is strongly wavelength-dependent. Over half the incoming total SW is near-infrared and is absorbed within the top metre or so. The visible band penetrates further, with blue light reaching deepest. The optical properties of the ocean are strongly affected by phytoplankton/chlorophyll, dissolved organic matter and suspended particulates.

MOM6 has a number of different schemes for defining how SW pentrates into the ocean interior.

### Beer–Lambert law

The Beer-Lambert law states that the loss of light intensity when it propagates in a medium is directly proportional to intensity and path length. This leads to an exponential attenuation law:

$\mathrm{SW}(d,b) = \mathrm{SW}(0,b) · \exp(−\kappa(d,b) \cdot d)$

The attenuation coefficient $\kappa$ is often referred to as the "optical density" or the "opacity". Larger $\kappa$ gives a shallower e-folding depth so is SW absorbed (and heats the ocean) closer to the surface.

The different opacity schemes in MOM6 provide different ways to define $\mathrm{SW}(0,b)$ and $\kappa(d,b)$.

## Implementation in MOM6

In MOM6, the implementation of the SW penetration is split into two parts:

1. Scheme-dependent calculation of the opacity and penetrating SW at the surface ($\kappa(d,b)$ and $\mathrm{SW}(0,b)$ above). This occurs in the `MOM_opacity` module, specifically in the `set_opacity` routine, which sets the following arrays:

  ```fortran
  type, public :: optics_type
    integer :: nbands
    real, allocatable :: opacity_band(:,:,:,:) ! <-- kappa(band, i, j, k)
    real, allocatable :: sw_pen_band(:,:,:)    ! <-- SW_0(band, i, j)
    ...
  end type optics_type
  ```
2. Scheme-independent application of the SW flux. This occurs in the `MOM_diabatic_aux` module, specifically in the `applyBoundaryFluxesInOut` routine, which
  - Accounts for the surface layer heating from the SW that does not penetrate (in `extractFluxes1d`);
  - Accounts for heating from penetrating SW by appling Beer-Lambert per band and through the column (in `absorbRemainingSW`). The heat added to a given layer is calculated from the convergence of the flux across all bands, i.e. the difference beween the total flux entering the top and leaving the bottom. Any unabsorbed SW at the bottom of the column is added as a uniform temperature increment across the column.

## MOM6 opacity schemes

MOM6 includes a number of schemes for calculating the opacity ($\kappa(d,b)$) and how much SW penetrates at the surface ($\mathrm{SW}(0,b)$). The scheme can be set in the `MOM_input` file using the `EXP_OPACITY_SCHEME` parameter (constant schemes; `VAR_PEN_SW = False`) or `OPACITY_SCHEME` parameter (chlorophyll-based schemes; `VAR_PEN_SW = True`). The number of wavelength bands and their bounds can be set using the `PEN_SW_NBANDS` and `OPACITY_BAND_WAVELENGTHS` parameters.

For the chlorophyll-based schemes, chlorophyll concentrations can be either provided via a file (2D and MOM uses the same value for all layers), or a 3D, prognostic field taken directly from a tracer package.

### `EXP_OPACITY_SCHEME = SINGLE_EXP`: constant, one band

User sets:
- `PEN_SW_SCALE`: the constant e-folding depth (i.e. $1/\kappa$)
- `PEN_SW_FRAC`: the constant fraction of total SW that penetrates the surface

### `EXP_OPACITY_SCHEME = DOUBLE_EXP`: constant, two bands

User sets:
- `PEN_SW_SCALE` and `PEN_SW_SCALE_2ND`: the constant e-folding depths for each band
- `SW_1ST_EXP_RATIO`: the constant fraction of total SW that penetrates the surface in the first band. The remaining SW penetrates in the second band.

### `OPACITY_SCHEME = MOREL_88`: chlorophyll-dependent, one band

Based on [Morel (1988)](https://agupubs.onlinelibrary.wiley.com/doi/10.1029/jc093ic09p10749) and [Morel & Antoine (1994)](https://journals.ametsoc.org/view/journals/phoc/24/7/1520-0485_1994_024_1652_hrwtuo_2_0_co_2.xml). The opacity and penetrating fraction are calculated from polynomial functions of chlorophyll concentration. Note this scheme allows specifying multiple bands, but in this case the opacaity and penetrating fraction are simply the same in each band.

User can set:
- `OPACITY_VALUES_MOREL`: coefficients to calculate the opacity from the chlorophyll
- `SW_PEN_FRAC_COEFS_MOREL`: coefficients to calculate the penetrating fraction of visible SW from the chlorophyll

### `OPACITY_SCHEME = OHLMANN_03`: chlorophyll-dependent, two bands

Based on [Ohlmann (2003)](https://journals.ametsoc.org/view/journals/clim/16/9/1520-0442_2003_16_1337_orhicm_2.0.co_2.xml), as implemented in CESM-POP. The opacity and penetrating fraction of total SW for each of the two bands are found using a look-up table in log10(chlorophyll) space.

Nothing else is user-configurable for this scheme — the lookup table coefficients are hardcoded.

### `OPACITY_SCHEME = MANIZZA_05`: chlorophyll-dependent, per-band

Based on [Manizza et al. (2005)](https://agupubs.onlinelibrary.wiley.com/doi/full/10.1029/2004GL020778). The opacity is calculated for each band from a polynomial function of chlorophyll concentration: $\kappa_b = a_b + b_b \cdot \mathrm{chl}^{c_b}$. By default, the first two bands use polynomial coefficients for blue and red light and any remaining bands are near-infrared with no chlorophyll-dependence. The penetrating shortwave is split into blue (band 1) and red (band 2) fractions of the visible shortwave. The near-infrared shortwave is divided evenly across any remaining bands.

User can set:
- `OPACITY_VALUES_MANIZZA`: pairs of coefficients ($a_b$ and $b_b$) for blue, red and near-infrared bands to calculate the opacity from the chlorophyll
- `CHOROPHYLL_POWER_MANIZZA`: powers ($c_b$) for blue, red and near-infrared bands to calculate the opacity from the chlorophyll
- `BLUE_FRAC_SW`: the fraction of visible SW to put in the blue band

