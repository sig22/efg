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
