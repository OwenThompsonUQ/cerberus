# Cerberus

A solver for the ideal five-moment multi-fluid plasma equations using the AMReX framework.

## Getting Started

The following repositories should all go into the same folder to make life easier when defining relative paths in the build procedure.

Download Cerberus:
```
git clone https://github.com/plasmasimuq/cerberus
```

The default branch is `develop`. Checkout a different branch with:
```
git checkout branch_name
```

Initialise the amrex submodule:
```
git submodule update --init
```
This will automatically pull the required version of amrex in `./cerberus/amrex`

### Prerequisites
Some external libraries are required to build the code. These can be installed on ubuntu systems with:
```
sudo apt install gfortran liblapack-dev libopenmpi-dev g++ libreadline-dev libboost-dev
```

### Building
The default makefile can be found at `cerberus/Exec/local/GNUmakefile`.
  - `DIM` is the dimensionality of the simulation. Set to 1, 2 or 3. Most of the examples run with `DIM = 2`.
  - `USE_EB = TRUE` will enable embedded boundaries. NOTE: this will only work for dimensionality >= 2.
  - `PROFILE` or `TINY_PROFILE` will enable profiling of the code. See the [AMReX documentation](https://amrex-codes.github.io/amrex/docs_html/AMReX_Profiling_Tools_Chapter.html) for more information.

Other options in this file are experimental.

To build the executable:

```
cd cerberus/Exec/local
make -f GNUmakefile -j4
``` 

Note that the `-j4` option is for multi-thread compilation and should be set to an appropriate value.

At this point you should have an executeable such as `MFP2d.gnu.MPI.ex` in ` cerberus/Exec/local`.

For a `DEBUG` version simply call 

```
make -f GNUmakefile DEBUG=TRUE -j4
```
Alternatively, set `DEBUG = TRUE` in the makefile.

### Running

The executeable can be run in parallel using `mpirun` and expects an `*.inputs` file that defines the simulation. This inputs file is made up of two parts; a section using the AMReX style configuration which defines the simulation infrastructure, and a Lua script which defines the physics side of the simulation including initial conditions and physical parameters.

The default Lua inputs file is found at `cerberus/Source/default.lua` which includes some helpful comments about each input argument.

Example problems may be found in `cerberus/Exec/testing`. 

To run a simulation simply call the executeable along with an inputs file. For example, using the Isotope-RMI test case:

```
cd Exec/testing/Isotope-RMI
mpirun -np 4 ../../local/MFP.2d.gnu.MPI.EB.PARTICLES.ex IRMI.inputs
```

Note the relative path to the executable. All output files will be saved to the location where the run command was issued. Also the `-np 4` option for `mpirun` dictates how many processes the code will be run on.

### Visualisation

Visualisation can be carried out using  [VisIt](https://wci.llnl.gov/simulation/computer-codes/visit/) or by using the provided python scripts in `cerberus/vis`.


### Eilmer

To expand the capabilities of the code it is possible to use the gas models implemented by [Eilmer](https://bitbucket.org/cfcfd/dgd-git/src/master/). This requires the download, compilation, and installation of the Eilmer gas library (found at `src/gas` in the Eilmer source) as well as the D-language compiler and runtime (see instructions [here](https://gdtk.uqcloud.net/)). To enable this capability within Cerberus the code must be compiled with the compile time option `EILMER_GAS=TRUE` along with appropriate directories provided for `DLANG_LIB` and `EILMER_LIB`. Note that the path to the gas model library is hard coded at compile time (see `Exec/Make.MFP`).

#### Using the Eilmer gas and reaction models
An example is provided in `Exec/testing/Eilmer-Gas`. This example shows how to prepare the files required by Eilmer using the `prep-gas` and `prep-chem` routines that are bundled in the Eilmer install. The gas model specified for a state must be of type `eilmer` and specify the name of the file that defines the gas model as generated by `prep-gas`. To define species within a state, the names and charges of each must be specified along with the mass fraction `alpha`. The specific heats and masses of the species are calculated from the gas model (make sure to define appropriate reference quantities). Note that  only `n-1` `alpha` values need to be defined, for `n` species, as the `n`-th value is calculated by assuming that all `alpha` values sum to unity. The reactions scheme is defined as an action, where the species named within the reactions scheme are assumed to be held by the associated states. Note that species can be defined across multiple states.

### Bug Reporting
Before creating a bug report, ensure you are on the latest commit of both `cerberus` and the `amrex` submodule as bugfixes happen frequently. To report a bug:
  - Open a new issue and apply the `bug` label from the options on the right.
  - Provide us with detailed instructions on how to recreate the bug.
  - Provide us with a minimal example that we can use to reproduce the bug.
  - Provide the generated Backtrace/output error messages.

We will not accept feature requests at this stage, as we already have a backlog of important features to implement.

We may provide *very* limited support for simple issues, however we do not guarantee support for problems you may be having with your specific input files.
