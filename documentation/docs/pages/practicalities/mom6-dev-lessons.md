# Lessons learned from MOM6 code development

Presenter: @jbisits (11/12/2025).

!!! tip
    Take-home message: when starting development, take care with which version of the MOM6 code repository you fork from.

## Background

When @jbisits started work on MOM6, he went to google and found the MOM6 codebase and then made a fork of `mom-ocean/MOM6` (the [central consortium repository](https://github.com/mom-ocean/MOM6)). Actually if one is looking to contribute to the MOM6 codebase _from_ Australia, there is an [access-nri/MOM6](https://github.com/access-nri/mom6) fork. They are different forks with different branches and code bases! 

## How do I find out which version of MOM6 I am currently using?

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

Specifically note that this line `access-om3/2025.08.001` highlights the [Spack bundle package](https://github.com/ACCESS-NRI/access-spack-packages/blob/408011d410ef43e54340537d030f7f93957774db/packages/access-om3/package.py) (specified [here](https://github.com/ACCESS-NRI/ACCESS-OM3/blob/7439c61acbe167abd79fc9be4dc333fadb9ac0cb/config/versions.json#L4)) and git tag that is being used for MOM6 code. One can then match this tag name `2025.08.001` from the model deployment repository, this link [lists all the tags](https://github.com/ACCESS-NRI/ACCESS-OM3/tags), and here is the tag we are looking for: [`2025.08.001`](https://github.com/ACCESS-NRI/ACCESS-OM3/releases/tag/2025.08.001) in this case.

Further information:

 - [MOM6 fork management for being a development node](https://github.com/ACCESS-NRI/MOM6/wiki);
 - [OM3 build system and deployment](https://access-om3-configs.access-hive.org.au/latest/infrastructure/Building/);
 - [NOAA-GFDL MOM6 development guide](https://github.com/NOAA-GFDL/MOM6-examples/wiki/Developers-guide);
 - [MOM6 development presentation](https://www.marshallward.org/mom6workshop/develop.html) by @marshallward.
