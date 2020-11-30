# DM-Sim: Quantum Circuit Simulator via Density Matrix on GPU Clusters

A Density Matrix Quantum Simulation Environment for Single-GPU/CPU, Single-Node-Multi-GPUs/CPUs and Multi-Nodes GPU/CPU Cluster. 

![alt text](img/example.png)

## Current version

Latest version: **2.2**

#### Version-2.2 Updates:
 - Add CPU backend for both OpenMP and MPI so that DM-Sim can support system without GPUs. Verified the C++/Python APIs, QDK/QIR on Summit.
 - Reform the source code file structure.

#### Version-2.1 Updates:
 - Support Q# code through [QIR](https://devblogs.microsoft.com/qsharp/introducing-quantum-intermediate-representation-qir/) and Bridge developed by Microsoft quantum team. The DM-Sim project is also listed as one of the [public projects](https://github.com/microsoft/qsharp-language/blob/main/Specifications/QIR/List.md) using QIR to support Q#/QDK. 
 - Enable the dump of current quantum circuit into a file stream.


### Version-2.0 Updates:
 - Provide python APIs, see the adder_n10_omp.py example in src.
 - Provide C++/CUDA APIs, see the adder_n10_omp.cu example in src.
 - Support single-GPU fast running (up to 14 qubits on a V100 GPU).
 - Support single-node-multi-GPU execution managed by OpenMP.
 - Support multi-node cluster execution managed by MPI and mpi4py.

If you're looking for the implementation we described in our SC-20 paper, please see the V1.0 release. DM-Sim is under active development. Please propose any bugs and suggest any features. We will continuously add new features. Questions and suggestions are welcome.

## About DM-Sim

Please see our SuperComputing (SC-20) [paper](doc/paper_sc20.pdf) for details. The paper is nominated for the best paper award in SC-20.

In this repository you will find a CUDA/C++ implementation for simulating deep quantum circuits on a single-GPU/CPU, a single-node-multi-GPUs/CPUs (e.g., NVIDIA [DGX-1](https://www.nvidia.com/en-gb/data-center/dgx-systems/dgx-1/), [DGX-2](https://www.nvidia.com/en-us/data-center/dgx-2/) and HGX)), and multi-nodes GPU/CPU cluster (like the [Summit supercomputer](https://www.olcf.ornl.gov/summit/) in ORNL) using full density matrices. Our DM_sim simulator fully supports [OpenQASM](https://github.com/Qiskit/openqasm) intermediate-representation (IR) language (see [spec](https://arxiv.org/pdf/1707.03429.pdf). OpenQASM can be generated by Qiskit, Cirq, ProjectQ and Scaffold (see below). It also supports Q#/QDK through [QIR](https://devblogs.microsoft.com/qsharp/introducing-quantum-intermediate-representation-qir/). For scale-up (i.e., single-node-multi-GPUs), we leverage fast intra-node interconnects such as NVLink, NV-SLI and NVSwitch (see our [benchmarking](https://drive.google.com/open?id=13S4zbl4asHlWv_oebHw1YOWkzDr6I9wg) paper and [evaluation](https://arxiv.org/abs/1903.04611) paper about several modern GPU Interconnect). This simulator is based on the Multi-GPU-BSP (MG-BSP) model, please see our SuperComputing-20 [paper](doc/paper_sc20.pdf) for details.

DM-Sim (OpenMP) simulates 1M general gates with 15-qubits gate-by-gate in 94 minutes on DGX-2 (16 NVIDIA V100 GPUs) using the density-operator -- on average 5.6 ms/gate. DM-Sim simulates a VQE-UCCSD 8-qubits circuit with 10808 gates in 249.3ms on a single NVIDIA V100 GPU -- on average 0.023 ms/gate.


## Supported Gate

|Gates | Meaning | Gates | Meaning |
|:---: | ------- | :---: | ------- |
|U3    | 3 parameter 2 pulse 1-qubit | CY | Controlled Y |
|U2 | 2 parameter 1 pulse 1-qubit  | SWAP | Swap     |
|U1 | 1 parameter 0 pulse 1-qubit  |  CH | Controlled H   |
|CX | Controlled-NOT  |  CCX | Toffoli   |
|ID | Idle gate or identity  |  CSWAP | Fredkin |
|X | Pauli-X bit flip  | CRX | Controlled RX rotation  |
|Y | Pauli-Y bit and phase flip  | CRY | Controlled RY rotation  |
|Z | Pauli-Z phase flip  |  CRZ | Controlled RZ rotation |
|H | Hadamard  |  CU1 | Controlled phase rotation  |
|S | sqrt(Z) phase  |  CU3 | Controlled U3 |
|SDG | conjugate of sqrt(Z)  |  RXX | 2-qubit XX rotation |
|T | sqrt(S) phase  | RZZ | 2-qubit ZZ rotation   |
|TDG | conjugate of sqrt(S) | RCCX | Relative-phase CXX |
|RX | X-axis rotation | RC3X | Relative-phase 3-controlled X |
|RY | Y-axis rotation | C3X | 3-controlled X |
|RZ | Z-axis rotation | C3XSQRTX | 3-controlled sqrt(X) |
|CZ | Controlled phase | C4X | 4-controlled X |
|W | W gate  |  RYY | 2-qubit YY rotation |
|C1 | Arbitrary 1-qubit gate | C2 | Arbitrary 2-qubit gate |

## Package Structure
#### **src**: DM-Sim source file
 - config.hpp: **configurations**, constants and macros.
 - Makefile: for compilation. 
 - util.cuh: GPU utility functions.
 - util_cpu.h: CPU utility functions.
 - dmsim_nvgpu_omp.cuh: major DM-Sim source file for 1-GPU or 1-node-N-GPUs with NVIDIA GPU backend.
 - dmsim_nvgpu_mpi.cuh: major DM-Sim source file for N-nodes with NVIDIA GPU backend.
 - dmsim_cpu_omp.hpp: major DM-Sim source file for 1-CPU_core or 1-node-N-CPU_cores with CPU backend.
 - dmsim_cpu_mpi.hpp: major DM-Sim source file for N-nodes with CPU backend.
 - py_nvgpu_omp_wrapper.cu: PyBind11 wrapper for Single/OpenMP python APIs with NVIDIA GPU backend.
 - py_nvgpu_mpi_wrapper.cu: PyBind11 wrapper for MPI python APIs via mpi4py with NVIDIA GPU backend.
 - py_cpu_omp_wrapper.cpp: PyBind11 wrapper for Single/OpenMP python APIs with CPU backend.
 - py_cpu_mpi_wrapper.cpp: PyBind11 wrapper for MPI python APIs via mpi4py with CPU backend.
 - adder_n10_cpu_omp.cpp: A 10-qubit adder example using C++ APIs (CPU backend).
 - adder_n10_cpu_mpi.cpp: A 10-qubit adder example using MPI C++ APIs (CPU backend).
 - adder_n10_nvgpu_omp.cu: A 10-qubit adder example using C++ APIs (NVIDIA GPU backend).
 - adder_n10_nvgpu_mpi.cu: A 10-qubit adder example using MPI C++/CUDA APIs (NVIDA GPU backend).
 - adder_n10_omp.py: A 10-qubit adder example using Python APIs for scaling-up (select backend inside).
 - adder_n10_mpi.py: A 10-qubit adder example using Python APIs for scaling-out (select backend inside).
 - qir_omp_wrapper.cu: It wrappers DM-Sim to realize QIR-Bridge API based on OpenMP (select backend inside).
 - qir_mpi_wrapper.cu: It wrappers DM-Sim to realize QIR-Bridge API based on MPI (select backend inside).
 - set_summit_qir_env.sh: The environment required for building Q#/QIR support on Summit.
 - vqe.ll: The example VQE QIR code.
 - vqe.qs: The example VQE Q# code.
 - vqe_omp_driver.cu: The driver code for the Q# based VQE example for scaling-up.
 - vqe_mpi_driver.cu: The driver code for the Q# based VQE example for scaling-out.

#### **benchmark**: 
 - OpenQASM-based benchmarks (.qasm), it contains some circuits in the paper, for more, please refer to our [QASMBench](https://github.com/uuudown/QASMBench).

#### **tool**: Supporting tools (will add support for other quantum languages).
 - dmsim_qasm.py: A script to convert OpenQASM code into a Python-API based python code (OpenMP version).
 - qelib1.inc: OpenQASM standard header file.
 - randomtest_n14.py: Test the single-GPU version using 100K Hadamard gates applying randomly on 14 qubits.
 - vqe_uccsd_n8.qasm: An OpenQASM based VQE circuit example generated from [Scaffold](https://github.com/epiqc/ScaffCC). 
 - run_dimsim_qasm.sh: Show how to convert the vqe_uccsd_n8.qasm circuit to the python script that can run on DM-Sim.
 - vqe_uccsd_n8.py: The generated python script that can run on DM-Sim.

#### **summit**: The files that are useful for running on ORNL Summit supercomputer
 - set_env.sh: set the environment for CUDA, C/C++ compiler, MPI, Python. We also describe how to setup the python-2.7 environment with pybind11 and mpi4py support on Summit. 
 - summit_dmsim.lsf: example lsf file for job submission on Summit (please update accordingly).
 - Summit.txt: System information about a node of Summit HPC generated using the SC [Author-Kit](https://github.com/SC-Tech-Program/Author-Kit) tool.

#### **artifact**: System configuration for the evaluation performed in our paper.
These are generated by using 
 - SLI.txt: For the SLI-system with two RTX2080 GPUs connected by NV-SLI bridge.
 - dgx-1P.txt: For the Pascal architecture P100-DGX-1 with 8 GPUs connected by NVLink-V1.
 - dgx-1V.txt: For the Volta architecture V100-DGX-1 with 8 GPUs connected by NVLink-V2.
 - dgx-2.txt: For the Volta architecture DGX-2 with 16 GPUs connected by NVSwitch.

#### **img**: images for the Repo.

## Configuration

You may need to update "src/Makefile" to configure your NVCC path and GPU architecture (e.g., -arch=sm_60 for P100, -arch=sm_70 for V100 and -arch=sm_80 for A100 GPUs). We need C++11 support (-std=c++11).  

```
CC = nvcc
FLAGS = -O3 -arch=sm_70 -std=c++11 -rdc=true
LIBS = -lm
```
## Prerequisite
DM-Sim requires the following packages.

|  Dependency  | Version |
|:-----------: | ------- |
|     CUDA     | 10.0 or later |
|  GCC (or XL) | 5.2 or later (16.01 for xlc)  |
|    OpenMP    | 4.0     |
| Spectrum-MPI | 10.3    | 
|  Python      | 2.7     |
|  Pybind11    | 2.5.0   |
|  mpi4py      | 3.0.3   | 

To build the scale-up version, we need OpenMP. To build the scale-out version, it needs MPI with GPUDirect support (we only tested using IBM XL and Spectrum-MPI on Summit). 

The QDK/QRI has additional dependency requirements. For ORNL Summit HPC, please check the setting file: set_summit_qir_env.sh

## Build

Please configure the Makefile for the targets, then use the following command for compilation: 
```text
make 
```

The default Python version is Python-2.7. If you are using the simulator in other python version, you can adjust accordingly in the Makefile. Note, if you need Python-3, say Python-3.7, you may need to take out the "-lpython3.7" from the compiler option before make.

## Execution

DM-Sim requires NVIDIA GPUs for execution. We have tested it on Tesla-P100 (Pascal, CC-6.0), Tesla-V100 (Volta, CC-7.0) and RTX2080 (Turing, CC-7.5). To run on scale-up workstations (e.g., DGX-1 and DGX-2), it needs all the GPUs to be directly connected by NVLink, NVSwitch or NV-SLI for all-to-all communication (when performing adjoint operation when transposing the density matrix)). Therefore, on DGX-1, it can use up to 4 GPUs (despite 8 in total) and provided they are directly interconnected, see our TPDS [Evaluation paper](https://arxiv.org/abs/1903.04611) on GPU interconnect for detail. For scale-out GPU clusters, it requires the support of [GPUDirect-RDMA](https://docs.nvidia.com/cuda/gpudirect-rdma/index.html) for direct GPU-memory access. On the ORNL Summit supercomputer, this can be enabled by *--smpiargs="-gpu"*. See the example [.lsf](src/summit_dmsim.lsf) file.

### Single GPU or single-node-multi-GPUs using C++/CUDA APIs

Writing a CUDA circuit code using DM-Sim C++/CUDA APIs can be simple:
```text
#include "util.cuh"
#include "gate_omp.cuh"
using namespace DMSim;

int main()
{
    int n_qubits = 10;
    int n_gpus = 4;
    sim.append(Simulation::X(0)); //add a Poly-X gate
    sim.append(Simulation::H(1)); //add a Hadamard gate
    sim.upload(); //upload to GPU
    sim.sim(); //simulate
    auto res = sim.measure(5); //measure with 5 repetitions
    print_measurement(res, 10, 5); //print results
}
```

When you have the circuit driver, compile and use the following command for execution:
```text
./adder_n10_omp
```

### Single GPU or single-node-multi-GPUs using Python APIs

Writing a python circuit code using DM-Sim C++/CUDA APIs can be even more simple:

```text
import dmsim_py_omp_wrapper as dmsim_omp
n_qubits = 10
n_gpus = 4
sim = dmsim_omp.Simulation(n_qubits, n_gpus))
sim.append(sim.X(0)) #add an X gate
sim.append(sim.H(1)) #add an H gate
sim.upload() #upload to GPU
sim.run() #run
sim.clear_circuit() #clear existing circuit
sim.append(sim.H(0)) #add a new H gate 
sim.upload() #upload to GPU
sim.run() #run new circuit on original states
res = sim.measure(10) #measure with 10 repetitions and return in a list
```

```text
python adder_n10_omp.py
```

### Scale-out
This is the execution command on ORNL Summit supercomputer (8 resource sets with 8 MPI ranks, 1 GPU per rank) with GPUDirect-RDMA enabled using Python APIs and C++/CUDA APIs. 
```text
jsrun -n8 -a1 -g1 -c1 --smpiargs="-gpu" python -m mpi4py adder_n10_mpi.py 10 
jsrun -n8 -a1 -g1 -c1 --smpiargs="-gpu" ./adder_n10_mpi
```
For the Python version, 10 means the number of qubits used. For the C++/CUDA version, it is written in the code.

## Expected Output

When build and execute , which realizes a ripple-carry adder using 10-qubits in total on a single-GPU, should print out the following output:
```text
============== DM-Sim ===============
nqubits:10, ngates:30, ngpus:4, comp:11.685 ms, comm:0.777 ms, sim:12.462 ms, mem:32.000 MB, mem_per_gpu:8.000 MB
=====================================

===============  Measurement (qubits=10, gates=30, tests=10) ================
Test-0: 1000000010
Test-1: 1000000010
Test-2: 1000000010
Test-3: 1000000010
Test-4: 1000000010
Test-5: 1000000010
Test-6: 1000000010
Test-7: 1000000010
Test-8: 1000000010
Test-9: 1000000010

```
The inputs are: carry-in cin = 0, A=0001, B=1111. The outputs are: B=B+A=0000, carry-out=1.

 - "**nqubits**" is the number of qubits simulated. 
 - "**ngates**" is the number of gates executed. 
 - "**ngpus**" is the number of GPUs utilized. 
 - "**comp**" is the computation time (ms) in the simulation. 
 - "**comm**" is the communication time (ms) in the simulation. 
 - "**sim**" is total simulation latency (ms). 
 - "**mem**" is the total GPU memory cost for all GPUs (in MBs). 
 - "**mem**" is the per-GPU memory usage (in MBs). 

The measurement measures all qubits at once. "**repetition**" refers to the number of repeated measurements. You can configure the number of trials when calling "measure()" in both C++/CUDA API and Python API. The default value is 10 times. 

## More Configurations

To simulate qubit-size larger than 15, the index is already larger than a normal unsigned integer, you need to define **IdxType** to "**unsigned long long**" in "config.hpp". The **ValType** is by default **double**.

When defining "CUDA_ERROR_CHECK", DM-Sim checks CUDA API error and kernel execution error.



## Performance
DM-Sim is bounded by GPU memory access bandwidth, and possibly by interconnect bandwidth. We use the [Roofline](https://en.wikipedia.org/wiki/Roofline_model) model to show the bound. The real sustainable bandwidth is profiled by using the [Roofline Toolkit](https://bitbucket.org/berkeleylab/cs-roofline-toolkit/src/master/) from LBNL. This following figure shows the Roofline model for the simulation on SLI, DGX-1P, DGX-1V and DGX-2 systems. See the files in the **[artifact](artifact)** folder. AI stands for arithmetic intensity for the DM simulation.
![alt text](img/roofline.png)

#### We show the performance of simulation by increasing the number of qubits (256 gates):
![alt text](img/qubits.png)

#### We show the performance of simulation by increasing the number of gates (14 qubits):
![alt text](img/qubits.png)

#### And performance bound on computation, memory access and communication:
![alt text](img/bound.png)

#### Performance for deep circuits on DGX-2 using 16 GPUs and 15 qubits using general 1-qubit gate(i.e., C1 gate):

|Gates | Computation | Communication | Simulation | Time/Gate |
|:---: | :---------: | :-----------: | :--------: | :-------: |
| 10K  |     53.8s   |     9.36ms    |   53.8s    |  5.38ms   |
| 100K |     558.0s  |     7.31ms    |   558.0s   |  5.58ms   |
|  1M  |    5645.5s  |     7.21ms    |   5645.5s  |  5.65ms   |

#### Performance on ORNL Summit supercomputer, the numbers on the bars indicate the number of GPUs utilized. For benchmarks, please see [QASMBench](https://arxiv.org/abs/2005.13018). Clearly, the communication overhead is much more significant than scale-up.
<img src="img/summit.png" width="500">

## Support Tools

#### dmsim_qasm.py
To translate an OpenQASM (e.g., vqe_uccsd_n8.qasm) to a DM-Sim python file (e.g., vqe_uccsd_n8.py):

```text
python dmsim_qasm.py -i vqe_uccsd_n8.qasm -o vqe_uccsd_n8.py
```
It outputs the target "vqe_uccsd_n8.py" and reports the number of qubits, the number of gates, and the number of CX/CNOT gates. Currently, it generates the OpenMP version python code.

```text
python dmsim_qasm_ass.py -i adder.qasm -o circuit.cuh -s omp
```


## More Benchmarks
We have developed an OpenQASM based benchmark suite called "[QASMBench](https://github.com/uuudown/QASMBench)" which provides more real quantum circuit benchmarks. Please see our [QASMBench](https://arxiv.org/abs/2005.13018) paper for details.


### OpenQASM

OpenQASM (Open Quantum Assembly Language) is a low-level quantum intermediate representation (IR) for quantum instructions, similar to the traditional *Hardware-Description-Language* (HDL) like Verilog and VHDL. OpenQASM is the open-source unified low-level assembly language for IBM quantum machines publically available on cloud that have been investigated and verified by many existing research works. Several popular quantum software frameworks use OpenQASM as one of their output-formats, including [Qiskit](https://github.com/Qiskit/qiskit), [Cirq](https://github.com/quantumlib/cirq), [Scaffold](https://github.com/epiqc/ScaffCC), [ProjectQ](https://github.com/ProjectQ-Framework/ProjectQ), etc.

#### Qiskit
The *Quantum Information Software Kit* ([Qiskit](https://github.com/Qiskit/qiskit)) is a quantum software developed by *IBM*. It is based on Python. OpenQASM can be generated from Qiskit via:
```text
QuantumCircuit.qasm()
```

#### Cirq
[Cirq](https://github.com/quantumlib/cirq) is a quantum software framework from *Google*. OpenQASM can be generated from Cirq (not fully compatible) via:
```text
cirq.Circuit.to_qasm()
```

#### Scaffold
[Scaffold](https://github.com/epiqc/ScaffCC) is a quantum programming language embedded in the C/C++ programming language based on the [LLVM](https://github.com/llvm/llvm-project) compiler toolchain. A Scaffold program can be compiled by [Scaffcc](https://arxiv.org/pdf/1507.01902.pdf) to OpenQASM via the "**-b**" compiler option.

#### ProjectQ
[ProjectQ](https://github.com/ProjectQ-Framework/ProjectQ) is a quantum software platform developed by *Steiger et al.* from ETH Zurich. The official website is [here](https://projectq.ch/). ProjectQ can generate OpenQASM when using IBM quantum machines as the backends:
```text
IBMBackend.get_qasm()
```

## Authors 

#### [Ang Li](http://www.angliphd.com/), Pacific Northwest National Laboratory (PNNL)

#### [Sriram Krishnamoorthy](https://hpc.pnl.gov/people/sriram/), Pacific Northwest National Laboratory (PNNL)

#### [Bo Fang](https://www.pnnl.gov/science/staff/staff_info.asp?staff_num=9511), Pacific Northwest National Laboratory (PNNL)

We are currently collaborating with Microsoft Quantum team (Alan Geller, Bettina Heim, Irina Yatsenko, Guen Prawiroatmodjo, Martin Roetteler) on improving the pipeline from Q# to [QIR](https://devblogs.microsoft.com/qsharp/introducing-quantum-intermediate-representation-qir/) to [DM-Sim](https://github.com/microsoft/qsharp-language/blob/main/Specifications/QIR/List.md). Many thanks to their strong support.


## Citation format

If you find DM-Sim useful, please cite our SC-20 paper:
 - Ang Li, Omer Subasi, Xiu Yang, and Sriram Krishnamoorthy. "Density Matrix Quantum Circuit Simulation via the BSP Machine on Modern GPU Clusters." In Proceedings of the International Conference for High Performance Computing, Networking, Storage and Analysis, 2020.

Bibtex:
```text
@inproceedings{li2020density,
    title={Density Matrix Quantum Circuit Simulation via the BSP Machine on Modern GPU Clusters},
    author={Li, Ang and Subasi, Omer and Yang, Xiu and Krishnamoorthy, Sriram},
    booktitle={Proceedings of the International Conference for High Performance Computing, Networking, Storage and Analysis},
    year={2020}
}
``` 


## License

This project is licensed under the BSD License, see [LICENSE](LICENSE) file for details.

## Acknowledgments

**PNNL-IPID: 31919-E, ECCN: EAR99, IR: PNNL-SA-143160**

This project is currently supported by the [Quantum Science Center (QSC)](https://qscience.org/). It was originally supported by PNNL's *Quantum Algorithms, Software, and Architectures* (QUASAR) LDRD Initiative. The Pacific Northwest National Laboratory (PNNL) is operated by Battelle for the U.S. Department of Energy (DOE) under contract DE-AC05-76RL01830. 

## Contributing

Please contact us If you'd like to contribute to DM-Sim. See the contact in our paper or my [webpage](http://www.angliphd.com).
