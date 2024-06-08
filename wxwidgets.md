# Opencv for MacOS M1
Instructions on how to build static libraries for arm64 on MacOS with Xcode 14.2...might work for others.


It took me about 3 days nonstop to get this, what should be a trivial task, to work. To make sure I don't lose this information when I eventually forget it, I'm writing it down here. Maybe some other poor soul coding on macs with xcode will find it.

Feel free to write if the same steps didn't work for you, or how you'd do it better.

## Prerequisites:
- Macbook M1 or newer
- Macports
- Xcode 14.2 probably similar to other versions.
- opencv 4.9.0
- You've completely given up and you can't figure it out.
- cmake +qt5
- eigen

### Instructions:
1. I assume you've given up and don't care about nuking ports. Remove all the old packages you had that are probably breaking cmake and compilers.
   - `sudo port -fp uninstall --follow-dependents installed`
   - `sudo port clean all`
   - `sudo port install cmake +qt5` The +qt5 is to have the cmake-gui command, but it wouldn't work for me either!
  
2. Create a directory for the source like `/home/user/cpp/libraries/source` the build and install directories
   - `mkdir source`
   - `mkdir build/arm64`
   - `mkdir opencv/arm64`
     
4. Download opencv source code using `git clone http://blah` in the source folder:
   ```
     cd source
     git clone link
     cd ..
     cd build/arm64
   ```
5. Use `ccmake` to setup the cmake configs linked to your source code directory while in the build directory.
   - `ccmake ~/source/opencv`
   - It will be blank probably, just hit `c`
   - Wait
   - hit `e`
   - You will get a giant list of things, they need to be adjusted according to your demands, but I left them all default except for the following (hit enter to change texts):
   - `CMAKE_OSX_ARCHITECTURES arm64` <-- filled in.
   - `EIGEN_INCLUDE_PATH /path/to/eigen/lib`
   - `Eigen3_DIR /path/to/eigen/lib`
   - `INSTALL_PATH /path/to/library/opencv/arm64`
   - `WITH_LAPACK OFF` from this issue https://github.com/opencv/opencv/issues/7970 (not required if adding Accelerate frame) https://github.com/opencv/opencv/issues/8086
   - When you are done hit `c` again. This may create errors, like fat binraries for java or ANT, just hit `c` again, and see if it goes away, adding the extra modules broke the build for some reason
   - If this manages to work generate the makefile, `g`
   - Time to compile. `cmake . && make -j8 all install` -j8 means compile with 8 cores, goes much faster.

6. Create a linker list
   - cd to your install directory `cd ~/opencv/arm64`
   - find all your libs you've made `find . -iname "lib*.a"`
     Terminal will spit out a list of the libraries it's created like this likely:
     ```
     ./lib/libopencv_photo.a
     ./lib/libopencv_videoio.a
     ./lib/libopencv_calib3d.a
     ./lib/libopencv_features2d.a
     ./lib/libopencv_dnn.a
      ```
   - And so on, copy these into some editor like vscode or something, and find/replace `./lib/lib` -> `-l` 
   - Find/replace `.a` -> blank
   - your linked libs should look like this then `-lopen_dnn` for example.
   - Copy this list and add `-L/directory/to/library/opencv/arm64/lib` at the top. (nospaces)

7. Start up Xcode
8. Create a new commandline project
9. Feel free to compile the hello world immediately, it sets up the products folder in the sidebar.
10. Once created go to General settings and add the following frameworks
    ```
    Core.Foundation
    Core.Graphics
    Appkit
    Cocoa
    OpenCL
    Quartzcore, and maybe just Quartz
    Accelerate (not required if not using LAPACK
    ```
11. Go to Build settings, search for **Architecture**
12. Under **Build Active Architecture Only** check yes if it's not already, make sure Debug and Release are also set.
13. Search for **Header Search Paths**
    - Add the path to the `/include/opencv4` directory under your install path, set to **recursive**
14. Search for **Library Search Paths**
    - Add the path to the `/lib/opencv4` directory under your install path, set to **recursive**
15. Search for **Other Linker Flags**
    - Add the premade list from step 6, it will show up in one line, then seperate to seperate ones, it's fine.
16. Search for `clang` or `llvm`, I use clang.
    - Under c++ language detect, set to c++17 std.
    - under C Languag detect, set to gnu11
17. Do whatever other crap you want.
18. Go to **Product -> Scheme -> Edit Scheme** Under Run, change to release and uncheck debug.
19. Add the following code to your main.cpp, and see if it works:
```c++
//
//  main.cpp
//  opencv_static_lib
//
//  Created by dr-mrsthemonarch on 08.01.24.
//
#include <iostream>
#include <opencv2/opencv.hpp>

std::string file;
cv::Mat inputImage;


int main(int argc, const char * argv[]) {
    // insert code here...
    
    std::cout << "Let's Find an image and import it.\n";
    std::cout << "Drag an image into terminal.:\n";
    std::cin >> file;
    inputImage = cv::imread(file);
    //test if image is empty
    if( inputImage.empty() ){
        std::cerr << "Didn't work guy!" << std::endl;
        return -1;
    }
    cv::imshow("Here's ya image guy", inputImage);
    cv::waitKey(0);
    cv::Mat gray;
    cv::cvtColor(inputImage, gray, cv::COLOR_BGR2GRAY);
    cv::imshow("And converted to Gray", gray);
    cv::waitKey(0);
    return 0;
}
```
If you're lucky, it somehow magically works...exactly this code and probably not any others. But _technically_ you have compiled yourself some static opencv libraries and can pass your compiled program to any other mac arm64 user without fail.

Good luck!
     
