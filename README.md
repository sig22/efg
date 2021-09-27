# About

This project provides an efficient implementation for traversing large compressed graphs on GPUs. Graphs are compressed with Elias-Fano encoding (on the CPU), and the breadth first search (BFS) implementation traverses such graphs on the GPU. Since GPU memory capacity is limited, this approach can accomodate larger graphs in device memory. Graphs are compressed by a factor of 1.5x on average relative to the compressed spare row (CSR) format. If the graphs still do not fit in memory, unified virtual memory (UVM) is used to transfer data at runtime from the host.

# Build Requirements

 - Ubuntu 18.04 or newer
 - [CUDA 11+](https://developer.nvidia.com/cuda-downloads)
 - [folly v2021.09.20.00](https://github.com/facebook/folly/releases/tag/v2021.09.20.00)


## Build

- `sudo apt install clang-10 libboost-all-dev libssl-dev libgoogle-glog-dev libdouble-conversion-dev`
- Download and extract the [folly tarball](https://github.com/facebook/folly/releases/tag/v2021.09.20.00). 
- Build and install folly at a specific path by calling  `./build.sh --scratch-path <scrath_path>`
- Clone this project. In the project's Makefile: 
    - Set `FOLLY_INSTALL_DIR` to <scratch_path\>/installed/folly
    - Set `FMT_INSTALL_DIR` to <scratch_path\>/installed/fmt-XXX, where XXX will be a string generated during the folly build process.
 - `make -j` 

## Input Graph Format
Input graphs need to be in the Compressed Sparse Row (CSR) format consisting of two binary files, `vertex.CSR` and `edge.CSR`. `vertex.CSR` contains row-offsets and is of length `|V| + 1`. `edge.CSR` contains column indices and is of length `|E|`.  Each neighbour list in `edge.CSR` should be in sorted order. The data type for the input graphs should be 64-bits. This is merely done to simplify the input handling and has no bearing on the compression. The reported compression ratio is relative to the optimal CSR size.

Small sample graphs are included in the repository under `sample_graphs`. These can be used for basic tests, but are meaningless for any performance measurements. Larger graphs are available at https://sig22.xyz/public/graphs/.

## Run

`./ef_bfs sample_graphs/tiny`

Usage:

    ./ef_bfs [OPTIONS] INPUT_DIR
    OPTIONS:
      -h [ --help ]           Print help
      -n [ --num ] arg (=100) Number of traversals
      -m [ --mapfile ] arg    Mapfile for remapping vertex ids. If provided, map[x]
                              will be used instead of x for starting a traversal
      -u [ --uvm ]            Use UVM for memory allocations
      -d [ --nosort ]         Disable the frontier sorting optimisation
      -r [ --root ] arg       Root for the traversal. Random roots will be used if 
                              unspecified

 - The `mapfile` option is useful for comparing the performance between reordered versions of the same graph. For example, the dataset above include graphs in their natural order and graphs reordered with HALO, and the reordered graphs include a mapfile. When running the the traversal on reordered graphs, the mapfile should be passed as an argument to get the same traversals.
 - The `-d` flag disables the partial frontier sorting optimisation. This should generally not be required, but it saves memory. It can be useful if the compressed graph barely fits in the GPU. For example, this is required for the `uk-2007-05`graph on a 12 GiB GPU.
 - The `-u`flag enables UVM allocations, which is useful for massive graphs that do not fit even after compression. DO NOT use `-d`along with `-u` as the performance will be very poor without the sorting optimisation in the UVM regime.
