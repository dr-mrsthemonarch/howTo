Here is a guide to setup geant4 for macOS and Clion.

First setup macport dependencies

```
sudo port install qt5
sudo port install xerces3
sudo port install zlib
```
Then download geant4, create some directories in your libraries folder, like source, install and build.

```
cd libraries
mkdir geant4
cd geant4
mkdir geant4_v11_build
mkdir genat4_v11_install
git clone https://github.com/Geant4/geant4.git
mv geant4 geant4_source
cd geant4_v11_build
```
Now you can setup the cmake file with the following code. This will enable qt5 and multithreading and set the program to release.
```
cmake -DCMAKE_INSTALL_PREFIX=/path/to/libraries/geant4/geant4_v11/geant4_v11_install  -DCMAKE_BUILD_TYPE=Release -DGEANT4_BUILD_MULTITHREADED=ON -DGEANT4_INSTALL_DATA=ON -DGEANT4_FORCE_QT4=OFF -DGEANT4_USE_QT_QT5=ON -DCMAKE_PREFIX_PATH="/opt/local/libexec/qt5" -DQt5_DIR=/opt/local/libexec/qt5/lib/cmake/Qt5 -DQT_QMAKE_EXECUTABLE=/opt/local/libexec/qt5/bin/qmake -DGEANT4_USE_QT=ON -DGEANT4_USE_GDML=ON -DGEANT4_USE_SYSTEM_EXPAT=ON -DGEANT4_USE_SYSTEM_ZLIB=ON -DGEANT4_BUILD_EXAMPLES=ON ../geant4_v11_source
```
Once this is done, you make, then install geant4. 
```
make -j$(nproc)
make install
```
Once this is done, ensure you also have the data sources downloaded from geant, either check the logs from compiling, or you can check installed share folder

```
ls geant4_install/share/Geant4/
```
you should see a data folder listed. 

Now you will need a valid CmakeList

```
cmake_minimum_required(VERSION 3.16)
project(BasicGeant4Test)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Set Geant4_DIR for custom installation
if(NOT DEFINED Geant4_DIR)
    set(Geant4_DIR "/Users/morbo/CLionProjects/libraries/geant4/geant4_v11/geant4_v11_install/lib/cmake/Geant4")
endif()

# Set MacPorts paths for dependencies
if(APPLE)
    list(APPEND CMAKE_PREFIX_PATH "/opt/local")
endif()

# Find required packages
find_package(Geant4 REQUIRED ui_all vis_all multithreaded)
find_package(Threads REQUIRED)

# Include Geant4 headers
include(${Geant4_USE_FILE})

# Include directories
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

# Source files
file(GLOB sources ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cc)
file(GLOB headers ${CMAKE_CURRENT_SOURCE_DIR}/include/*.hh)

# Add executable
add_executable(basicTest main.cc ${sources} ${headers})

# Set output directory for executable
set_target_properties(basicTest PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/executable
)

# Copy macro files to the executable directory
configure_file(${CMAKE_SOURCE_DIR}/init_vis.mac ${CMAKE_CURRENT_SOURCE_DIR}/executable/init_vis.mac COPYONLY)
configure_file(${CMAKE_SOURCE_DIR}/gui.mac ${CMAKE_CURRENT_SOURCE_DIR}/executable/gui.mac COPYONLY)
configure_file(${CMAKE_SOURCE_DIR}/init_vis.mac ${CMAKE_CURRENT_SOURCE_DIR}/executable/run.mac COPYONLY)
configure_file(${CMAKE_SOURCE_DIR}/gui.mac ${CMAKE_CURRENT_SOURCE_DIR}/executable/run_mt.mac COPYONLY)

# Link libraries
target_link_libraries(basicTest ${Geant4_LIBRARIES} Threads::Threads)

# Optimization flags for Release builds
if(CMAKE_BUILD_TYPE STREQUAL "Release")
    target_compile_options(basicTest PRIVATE -O3 -DNDEBUG)
endif()
```

Afterwards, you will also have to create a release profile with some settings in clion, to go settings -> cmake click the plus, create a release profile and in cmake options at the following:

```
-DCMAKE_BUILD_TYPE=Release
-DGeant4_DIR=/path/to/libraries/geant4/geant4_v11/geant4_v11_install/lib/cmake/Geant4
-DZLIB_ROOT=/opt/local
-DZLIB_LIBRARY=/opt/local/lib/libz.dylib
-DZLIB_INCLUDE_DIR=/opt/local/include
```

and under Environment add

```
export G4VIS_DEFAULT_DRIVER=Qt3D
```

Now you can see my other repo for example geant4 code.
