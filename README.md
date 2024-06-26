# Vector Particle-In-Cell (VPIC) Project [Forked]

Welcome to the legacy version of VPIC! The new version of VPIC, based on the
Kokkos performance portable framework, is available here:
[https://github.com/lanl/vpic-kokkos](https://github.com/lanl/vpic-kokkos).
This legacy version is no longer under active development, and new users are
encouraged to use the Kokkos version.

VPIC is a general purpose particle-in-cell simulation code for modeling
kinetic plasmas in one, two, or three spatial dimensions. It employs a
second-order, explicit, leapfrog algorithm to update charged particle
positions and velocities in order to solve the relativistic kinetic
equation for each species in the plasma, along with a full Maxwell
description for the electric and magnetic fields evolved via a second-
order finite-difference-time-domain (FDTD) solve. The VPIC code has been
optimized for modern computing architectures and uses Message Passing
Interface (MPI) calls for multi-node application as well as data
parallelism using threads. VPIC employs a variety of short-vector,
single-instruction-multiple-data (SIMD) intrinsics for high performance
and has been designed so that the data structures align with cache
boundaries. The current feature set for VPIC includes a flexible input
deck format capable of treating a wide variety of problems. These
include: the ability to treat electromagnetic materials (scalar and
tensor dielectric, conductivity, and diamagnetic material properties);
multiple emission models, including user-configurable models; arbitrary,
user-configurable boundary conditions for particles and fields; user-
definable simulation units; a suite of "standard" diagnostics, as well
as user-configurable diagnostics; a Monte-Carlo treatment of collisional
processes capable of treating binary and unary collisions and secondary
particle generation; and, flexible checkpoint-restart semantics enabling
VPIC checkpoint files to be read as input for subsequent simulations.
VPIC has a native I/O format that interfaces with the high-performance
visualization software Ensight and Paraview. While the common use cases
for VPIC employ low-order particles on rectilinear meshes, a framework
exists to treat higher-order particles and curvilinear meshes, as well
as more advanced field solvers.

# Attribution

Researchers who use the VPIC code for scientific research are asked to cite
the papers by Kevin Bowers listed below.

1. Bowers, K. J., B. J. Albright, B. Bergen, L. Yin, K. J. Barker and
   D. J. Kerbyson, "0.374 Pflop/s Trillion-Particle Kinetic Modeling of
   Laser Plasma Interaction on Road-runner," Proc. 2008 ACM/IEEE Conf.
   Supercomputing (Gordon Bell Prize Finalist Paper).
   http://dl.acm.org/citation.cfm?id=1413435

2. K.J. Bowers, B.J. Albright, B. Bergen and T.J.T. Kwan, Ultrahigh
   performance three-dimensional electromagnetic relativistic kinetic
   plasma simulation, Phys. Plasmas 15, 055703 (2008);
   http://dx.doi.org/10.1063/1.2840133

3. K.J. Bowers, B.J. Albright, L. Yin, W. Daughton, V. Roytershteyn,
   B. Bergen and T.J.T Kwan, Advances in petascale kinetic simulations
   with VPIC and Roadrunner, Journal of Physics: Conference Series 180,
   012055, 2009

# Getting the Code

To checkout the forked VPIC source, do the following:

```bash
    git clone https://github.com/mavhawk64/vpic.git
```

## Branches

The stable release of vpic exists on `master`, the default branch.

For more cutting edge features, consider using the `devel` branch.

User contributions should target the `devel` branch.

# Requirements

The primary requirement to build VPIC is a C++11 capable compiler and
an up-to-date version of MPI.

# Build Instructions

```bash
    cd vpic
```

VPIC uses the CMake build system. To configure a build, do the following from
the top-level source directory:

```bash
    mkdir build
    cd build
```

The `./arch` directory also contains various cmake scripts (including specific build options) which can help with building, but the user is left to select which compiler they wish to use. The scripts are largely organized into folders by compiler, with specific flags and options set to match the target compiler.

Any of the arch scripts can be invoked specifying the file name from inside a build directory:

```bash
    ../arch/reference-Debug
```

After configuration, simply type:

```bash
    make
```

# MacOS: Building an example input deck

```bash
    . /path/to/vpic/build/bin/vpic_mac /path/to/vpic/sample/harris
```

Replace `/path/to/vpic` with the path to the VPIC source directory.

Otherwise, you may use the relative path from the VPIC source directory:

```bash
    . ./build/bin/vpic_mac ./sample/harris
```

If you do not want the script to change to the `outputs/$deck` directory,
you can run either of the following commands:

```bash
    /path/to/vpic/build/bin/vpic_mac /path/to/vpic/sample/harris
```


```bash
    ./build/bin/vpic_mac ./sample/harris
```

(Notice the lack of the source (.) command in the above commands.)

`vpic_mac` is a script similar to the `vpic` script, but it adds some
additional functionality where it changes to the `outputs/$deck` directory
(where _$deck_ is _harris_ for the example) after compiling the code to
_$deck.Linux_ (_harris.Linux_ for the example). Finally, it runs the _$deck.Linux_
executable with the following command (relative to the `outputs/$deck` directory):

```bash
    mpirun -np 4 $deck.Linux
```

# ORIGINAL: Building an example input deck

After you have successfully built VPIC, you should have an executable in
the `bin` directory called `vpic` (`./bin/vpic`). To build an executable from one of
the sample input decks (found in `./sample`), simply run:

```bash
    ./bin/vpic input_deck
```

where _input_deck_ is the name of your sample deck. For example, to build
the _harris_ input deck in the _sample_ subdirectory
_(assuming that your build directory is located in the top-level
source directory)_:

```bash
    ./bin/vpic ../sample/harris
```

Beginners are advised to read the harris deck thoroughly, as it provides many examples of common uses cases.

# Command Line Arguments

Note: Historic VPIC users should note that the format of command line arguments was changed in the first open source release. The equals symbol is no longer accepted, and two dashes are mandatory.

In general, command line arguments take the form `--command value`, in which two dashes are followed by a keyword, with a space delimiting the command and the value.

The following specific syntax is available to the users:

## Threading

Threading (per MPI rank) can be enabled using the following syntax:

```bash
    ./binary.Linux --tpp n
```

Where n specifies the number of threads

### Example:

```bash
    mpirun -n 2 ./binary.Linux --tpp 2
```

To run with VPIC with two threads per MPI rank.

## Checkpoint Restart

VPIC can restart from a checkpoint dump file, using the following syntax:

```bash
    ./binary.Linux --restore <path to file>
```

### Example:

```bash
    ./binary.Linux --restore ./restart/restart0
```

To restart VPIC using the restart file `./restart/restart0`

# Compile Time Arguments

Currently, the following options are exposed at compile time for the users consideration:

## Particle Array Resizing

- `DISABLE_DYNAMIC_RESIZING` (default `OFF`): Enable to disable the use of dynamic particle resizing
- `SET_MIN_NUM_PARTICLES` (default 128 [4kb]): Set the minimum number of particles allowable when dynamically resizing

## Threading Model

- `USE_PTHREADS`: Use Pthreads for threading model, (default `ON`)
- `USE_OPENMP`: Use OpenMP for threading model

## Vectorization

The following CMake variables are used to control the vector implementation that
VPIC uses for each SIMD width. Currently, there is support for 128 bit, 256 bit
and 512 bit SIMD widths. The default is for each of these CMake variables to be
disabled which means that an unvectorized reference implementation of functions
will be used.

- `USE_V4_SSE`: Enable 4 wide (128-bit) SSE
- `USE_V4_AVX`: Enable 4 wide (128-bit) AVX
- `USE_V4_AVX2`: Enable 4 wide (128-bit) AVX2
- `USE_V4_ALTIVEC`: Enable 4 wide (128-bit) Altivec
- `USE_V4_PORTABLE`: Enable 4 wide (128-bit) portable implementation

- `USE_V8_AVX`: Enable 8 wide (256-bit) AVX
- `USE_V8_AVX2`: Enable 8 wide (256-bit) AVX2
- `USE_V8_PORTABLE`: Enable 8 wide (256-bit) portable implementation

- `USE_V16_AVX512`: Enable 16 wide (512-bit) AVX512
- `USE_V16_PORTABLE`: Enable 16 wide (512-bit) portable implementation

Several functions in VPIC have vector implementations for each of the three SIMD
widths. Some only have a single implementation. An example of the latter is
move_p which only has a reference implementation and a V4 implementation.

It is possible to have a single CMake vector variable configured as ON for each
of the three supported SIMD vector widths. It is recommended to always have a
CMake variable configured as ON for the 128 bit SIMD vector width so that move_p
will be vectorized. In addition, it is recommended to configure as ON the CMake
variable that is associated with the native SIMD vector width of the processor
that VPIC is targeting. If a CMake variable is configured as ON for each of the
three available SIMD vector widths, then for a given function in VPIC, the
implementation which supports the largest SIMD vector length will be chosen. If
a V16 implementation exists, it will be chosen. If a V16 implementation does not
exist but V8 and V4 implementations exist, the V8 implementation will be chosen.
If V16 and V8 implementations do not exist but a V4 implementation does, it will
be chosen. If no SIMD vector implementation exists, the unvectorized reference
implementation will be chosen.

In summary, when using vector versions on a machine with 256 bit SIMD, the
V4 and V8 implementations should be configured as ON. When using a machine
with 512 bit SIMD, V4 and V16 implementations should be configured as ON.
When choosing a vector implementation for a given SIMD vector length, the
implementation that is closest to the SIMD instruction set for the targeted
processor should be chosen. The portable versions are most commonly used for
debugging the implementation of new intrinsics versions. However, the portable
versions are generally more performant than the unvectorized reference
implemenation. So, one might consider using the V4_PORTABLE version on ARM
processors until a V4_NEON implementation becomes available.

## Output

- `VPIC_PRINT_MORE_DIGITS`: Enable more digits in timing output of status reports

## Particle sorting implementation

The CMake variable below allows building VPIC to use the legacy, thread serial
implementation of the particle sort algorithm.

- `USE_LEGACY_SORT`: Use legacy thread serial particle sort, (default `OFF`)

The legacy particle sort implementation is the thread serial particle sort
implementation from the legacy v407 version of VPIC. This implementation
supports both in-place and out-of-place sorting of the particles. It is very
competitive with the thread parallel sort implementation for a small number
of threads per MPI rank, i.e. 4 or less, especially on KNL because sorting
the particles in-place allows the fraction of particles stored in High
Bandwidth Memory (HBM) to remain stored in HBM. Also, the memory footprint
of VPIC is reduced by the memory of a particle array which can be significant
for particle dominated problems.

The default particle sort implementation is a thread parallel implementation.
Currently, it can only perform out-of-place sorting of the particles. It will
be more performant than the legacy implementation when using many threads per
MPI rank but uses more memory because of the out-of-place sort.

# Workflow

Contributors are asked to be aware of the following workflow:

1. Pull requests are accepted into `devel` upon tests passing
2. `master` should reflect the _stable_ state of the code
3. Periodic releases will be made from `devel` into `master`

# Feedback

Feedback, comments, or issues can be raised through [GitHub issues](https://github.com/lanl/vpic/issues).

A mailing list for open collaboration can also be found [here](https://groups.google.com/forum/#!forum/vpic-users)

# Versioning

Version release summary:

## V1.2 (October 2020)

- Improved Neon intrinsics support
- Added Takizuka-Abe collision operator
- Threaded hydro_p pipelines
- Added unit documentation

## V1.1 (March 2019)

- Added V8 and V16 functionality
- Improved documentation and build processes
- Significantly improved testing and correctness capabilities

## V1.0

Initial release

# Release

This software has been approved for open source release and has been assigned **LA-CC-15-109**.

# Copyright

© (or copyright) 2020. Triad National Security, LLC. All rights reserved. This
program was produced under U.S. Government contract 89233218CNA000001 for Los
Alamos National Laboratory (LANL), which is operated by Triad National
Security, LLC for the U.S. Department of Energy/National Nuclear Security
Administration. All rights in the program are reserved by Triad National
Security, LLC, and the U.S. Department of Energy/National Nuclear Security
Administration. The Government is granted for itself and others acting on its
behalf a nonexclusive, paid-up, irrevocable worldwide license in this material
to reproduce, prepare derivative works, distribute copies to the public,
perform publicly and display publicly, and to permit others to do so.

# License

VPIC is distributed under a [BSD license](LICENSE.md).
