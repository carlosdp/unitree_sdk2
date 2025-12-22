# unitree_sdk2
Unitree robot sdk version 2.

### Prebuild environment
* OS  (Ubuntu 20.04 LTS)  
* CPU  (aarch64 and x86_64)   
* Compiler  (gcc version 9.4.0) 

### Build examples

To build the examples inside this repository:

```bash
mkdir build
cd build
cmake ..
make
```

### Installation

To build your own application with the SDK, you can install the unitree_sdk2 to your system directory:

```bash
mkdir build
cd build
cmake ..
sudo make install
```

Or install unitree_sdk2 to a specified directory:

```bash
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/opt/unitree_robotics
sudo make install
```

You can refer to `example/cmake_sample` on how to import the unitree_sdk2 into your CMake project. 

Note that if you install the library to other places other than `/opt/unitree_robotics`, you need to make sure the path is added to "${CMAKE_PREFIX_PATH}" so that cmake can find it with "find_package()".

### Python wheels (Linux)

The `python_binding` target is packaged with `cibuildwheel` to produce manylinux2014 wheels for `x86_64` and `aarch64` (compatible with Ubuntu 20.04/22.04, including Jetson boards running JetPack 5/6). The wheels bundle the CycloneDDS runtime so no extra system packages are required.

Build locally:

```bash
pip install build
python -m build --wheel
```

GitHub Actions (`.github/workflows/c-cpp.yml`) publishes wheel artifacts for both architectures on every push/PR.

### Notice
For more reference information, please go to [Unitree Document Center](https://support.unitree.com/home/zh/developer).
