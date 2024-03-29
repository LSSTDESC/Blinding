[![DESC Blinding](https://github.com/LSSTDESC/Blinding/actions/workflows/CI.yml/badge.svg)](https://github.com/LSSTDESC/Blinding/actions/workflows/CI.yml)
[![codecov](https://codecov.io/gh/LSSTDESC/Blinding/graph/badge.svg?token=T3L9QM4PTT)](https://codecov.io/gh/LSSTDESC/Blinding)
[![LSST DESC Blinding Slack](https://img.shields.io/badge/join-Slack-4A154B)](https://lsstc.slack.com/archives/CT14ZF2AH)

# DESC Blinding Modules
This repostory (under development) contains the blinding modules for blinding at the following levels of the analysis:
- Data-vector measurements
- Posterior distribution [not yet developed]
- (TBC) Catalogues

Other info: the previous README can be found [here](bckp_README.md)

> :warning: Important notice :warning: : the term "blinding" and the name "smokescreen" are used as placeholders and are not final. The final name will be decided by the DESC Blinding Working Group in discussions with the collaboration.

## Installation
**Creating a new environment:**

You can create a new conda environment with the required packages using the `environment.yml` file:
```bash
conda env create -f environment.yml
```
This will create a new environment called `desc_blinding` with the required packages. You can activate the environment with:
```bash
conda activate desc_blinding
```

**Using an existing environment:**

If you want to install the `blinding` package in an existing environment, you can install it using:
```bash
conda activate myenv
conda env update --name myenv --file environment.yml --prune
```

After installing the dependencies from `environment.yml`, you can install the `blinding` package using:
```bash
python -m pip install [-e] .
```
The `-e` flag is optional and installs the package in editable mode (useful for development).

### Testing the installation
You can test the installation by running the tests:
```bash
pytest .
```

## Usage: Data-vector Blinding
### Likelihood Requirements
The blinding module requires the Firecrown likelihoods to be built with certain requirements. First we bust be able to build the likelihoods providing a [sacc](https://github.com/LSSTDESC/sacc/tree/master) object with the measurements for the data-vector:
```python
def build_likelihood(build_parameters):
    """
    This is a generic likelihood theory model 
    for a generic data vector.
    """
    sacc_data = build_parameters['sacc_data']
```
(This is simular to what is currently done in [TXPipe](https://github.com/LSSTDESC/TXPipe/blob/df0dcc8c1e974576dd1942624ab5ff7bd0fbbaa0/txpipe/utils/theory_model.py#L19))

The likelihood module also must have a method `.compute_theory_vector(ModellingTools)` which calls for the calculation of the theory vector inside the likelihood object. 

The likelihood can be provided either as a path to the python file containing the `build_likelihood` function or as a python module. In the latter case, the module must be imported.

### Blinding a data vector
TL;DR: Check the `notebooks/test_blinding_prototype.ipynb` for a working example using the 
`des_y1_3x2pt_PT` [example from firecrown](https://github.com/LSSTDESC/firecrown/tree/master/examples/des_y1_3x2pt).

The blinding module can be used to blind the data-vector measurements. The module can be used as follows:
```python
# import the module
import pyccl as ccl
from blind import Smokescreen
# import the likelihood that contains the model and data vector
[...]
import my_likelihood

# create the cosmology ccl object
cosmo = ccl.Cosmology(Omega_c=0.27, 
                      Omega_b=0.045, 
                      h=0.67, 
                      sigma8=0.8, 
                      n_s=0.96, 
                      transfer_function='bbks')
# load a sacc object with the data vector [FIXME: this is a placeholder, the sacc object should be loaded from the likelihood]
sacc_data = sacc.Sacc.load_fits('path/to/data_vector.sacc')
# create a dictionary of the necessary firecrown nuisance parameters
syst_dict = {
            "ia_a_1": 1.0,
            "ia_a_2": 0.5,
            "ia_a_d": 0.5,
            "lens0_bias": 2.0,
            "lens0_b_2": 1.0,
            "lens0_b_s": 1.0,
            "lens0_mag_bias": 1.0,
            "src0_delta_z": 0.000,
            "lens0_delta_z": 0.000,}
# create the smokescreen object
smkscr = Smokescreen(cosmo, syst_dict, sacc_data, my_likelihood, 
                    {'Omega_c': (0.22, 0.32), 'sigma8': (0.7, 0.9)})
# blind the data vector
smkscr.calculate_blinding_factor()
blinded_dv = smkscr.apply_blinding_to_likelihood_datavec()
```


## Legacy Blinding
Legacy Blinding scripts for 2pt data vector blinding with Cosmosis moved to a [new repository](https://github.com/LSSTDESC/legacy_blinding).

The [Legacy Blinding](https://github.com/LSSTDESC/legacy_blinding) repositories contains an updated version of Jessie Muir's scripts for blinding data-vectors using Cosmosis. The scripts have been adapted to work as a standalone module with Cosmosis V2 and [SACC](https://sacc.readthedocs.io/en/latest/) [Under Construction!].

> Under development. For questions contact @arthurmloureiro, @jessmuir, or @jablazek
