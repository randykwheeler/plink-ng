# Building plink-ng in WSL (Windows)

This document outlines the steps taken to successfully compile the `plink-ng` (specifically the `2.0` version) application on Windows using Windows Subsystem for Linux (WSL).

## Prerequisites

1. **WSL:** Ensure WSL is installed and running a Linux distribution (e.g., Ubuntu).
2. **Basic Build Tools:** You need standard compilation tools.

## Step-by-Step Instructions

1. **Clone the Repository**
    First, clone the source code from the official GitHub repository:

    ```bash
    git clone https://github.com/chrchang/plink-ng.git
    cd plink-ng
    ```

2. **Install Dependencies**
    The `plink-ng` 2.0 dynamic build requires a few specific libraries for compilation, including standard C/C++ compilation tools, `zlib`, `openblas`, and `lapacke`.

    You can install all of these prerequisites directly within your WSL environment using your package manager (like `apt` on Ubuntu/Debian):

    ```bash
    sudo apt-get update
    sudo apt-get install -y build-essential zlib1g-dev libopenblas-dev liblapacke-dev
    ```

3. **Navigate to the Build Directory**
    The build configurations are separated depending on your target system. Since we are in WSL (a standard Linux environment), we'll use the dynamic Linux build instructions:

    ```bash
    cd 2.0/build_dynamic
    ```

4. **Compile the Code**
    Run the `make` command to start the build process. This will compile the various components and link the libraries.

    ```bash
    make
    ```

## Output

Once the compilation finishes without errors, you will find the compiled `plink2` executable (and potentially `pgen_compress`) securely built in your directory!
