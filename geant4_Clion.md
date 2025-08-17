#Here is a guide to setup geant4 for macOS and Clion.

First setup macport dependencies

```
sudo port install qt5
sudo port install xerces3
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
Now you can setup the cmake file with the following code. This will enable qt5 and multithreading
```
cmake -DCMAKE_INSTALL_PREFIX=/Users/morbo/CLionProjects/libraries/geant4/geant4_v11/geant4_v11_install  -DCMAKE_BUILD_TYPE=Release -DGEANT4_BUILD_MULTITHREADED=ON -DGEANT4_INSTALL_DATA=ON -DGEANT4_FORCE_QT4=OFF -DGEANT4_USE_QT_QT5=ON -DCMAKE_PREFIX_PATH="/opt/local/libexec/qt5" -DQt5_DIR=/opt/local/libexec/qt5/lib/cmake/Qt5 -DQT_QMAKE_EXECUTABLE=/opt/local/libexec/qt5/bin/qmake -DGEANT4_USE_QT=ON -DGEANT4_USE_GDML=ON -DGEANT4_USE_SYSTEM_EXPAT=ON -DGEANT4_USE_SYSTEM_ZLIB=ON -DGEANT4_BUILD_EXAMPLES=ON ../geant4_v11_source
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

