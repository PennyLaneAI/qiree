# QIR-EE with Lightning simulator backend

The [PennyLane-Lightning](https://github.com/PennyLaneAI/pennylane-lightning) plugins are high-performance quantum simulators, which are part of the [PennyLane](https://github.com/PennyLaneAI/pennylane) ecosystem. The simulators include the following backends (which can be used with QIREE):
- `lightning.qubit`: a fast state-vector simulator with optional OpenMP additions and parallelized gate-level SIMD kernels.
- `lightning.gpu`: a state-vector simulator based on the NVIDIA cuQuantum SDK.
- `lightning.kokkos`: a state-vector simulator written with Kokkos. It can exploit the inherent parallelism of modern processing units supporting the OpenMP, CUDA or HIP programming models.

## Installing a Lightning simulator

More information on installing Pennylane Lightning simulators from source, please visit the [Lightning installation page](https://docs.pennylane.ai/projects/lightning/en/latest/dev/installation.html). Note: QIREE is tested to work with PennyLane Lightning simulators v0.42.

### Quick start

The easiest way to get started is to install a Lightning simulator (`pennylane-lightning`/`pennylane-lightning-gpu`/`pennylane-lightning-kokkos`) from PyPI via pip:

```
$ pip install pennylane-lightning-kokkos==0.42.0

$ pip show pennylane-lightning-kokkos
Name: PennyLane_Lightning_Kokkos
Version: 0.42.0
Summary: PennyLane-Lightning plugin
Home-page: https://github.com/PennyLaneAI/pennylane-lightning
Author:
Author-email:
License: Apache License 2.0
Location: <site packages path>
Requires: pennylane, pennylane-lightning
```
Running `pip install pennylane` or `pip install pennylane-lightning` will automatically install the `lightning.qubit` (CPU) simulator, and other simulators can be installed by running `pip install pennylane-lightning-kokkos / pennylane-lightning-gpu`. Note: by default, the pre-built `lightning.kokkos` wheels from pip are built with Kokkos OpenMP enabled for CPU. To build Kokkos for other devices (e.g. CUDA or HIP GPUs), please install from source. Instruction can be found [here](https://docs.pennylane.ai/projects/lightning/en/latest/lightning_kokkos/installation.html).

When installing Pennylane-Lightning from pip or from source, you will have the shared libraries for each of the simulator installed. These are named `liblightning_qubit_catalyst.so`/`liblightning_kokkos_catalyst.so`/`liblightning_GPU_catalyst.so` respectively.

To obtain the path to the library:
```
$ export PL_PATH=$(python -c "import site; print( f'{site.getsitepackages()[0]}/pennylane_lightning')")

$ ls $PL_PATH
... liblightning_qubit_catalyst.so  liblightning_kokkos_catalyst.so ...
```

You can swap `pennylane-lightning-kokkos` for `pennylane-lightning-gpu` for lightning.gpu and `pennylane-lightning` for lightning.gpu simulators.

## Compile QIR-EE with Lightning backend

To compile QIR-EE with lightning backend:

```
# Set the path for the lightning simulator shared library
export LIGHTNING_SIM_PATH=$(python -c "import site; print( f'{site.getsitepackages()[0]}/pennylane_lightning')")/liblightning_kokkos_catalyst.so

# Proceed with usual build instructions, but with `-DQIREE_USE_LIGHTNING=ON` cmake flag
cd qiree/
mkdir build; cd build
cmake -DQIREE_USE_LIGHTNING=ON ..
make

```

Note:
- replace `libligghtning_kokkos_catalyst.so` with `liblightning_qubit_catalyst.so` or `liblightning_GPU_catalyst.so` if required.
- when running with `lightning.gpu` simulator for Nvidia GPUs, include `cuquantum` libraries in the library path (which will be installed as a dependency from Python), i.e.

```
LD_LIBRARY_PATH=$(python -c "import site; print( f'{site.getsitepackages()[0]}/cuquantum')")/lib:$LD_LIBRARY_PATH ./test_rt_device.out
```


## Running the example

To run (in the `build` directory):

```
$ ./bin/qir-lightning ../examples/bell.ll -s 100
{"00":43,"11":57}
```
