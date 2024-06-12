# wxwidgets for MacOS M1 and Xcode 14+
Instructions on how to build static libraries for arm64 on MacOS with Xcode 14.2...might work for others.


It took me some hours to get this, what should be a trivial task, to work. To make sure I don't lose this information when I eventually forget it, I'm writing it down here. Maybe some other poor soul coding on macs with xcode will find it.

Feel free to write if the same steps didn't work for you, or how you'd do it better.

## Prerequisites:
- Macbook M1 or newer
- Macports
- Xcode 14.2 probably similar to other versions.
- wxwidgets 3.2.x

### Instructions:
1. Download wxwidgets from: https://github.com/wxWidgets/wxWidgets/releases
  
2. Create a directory for the source like `/home/user/cpp/libraries/source` the build and install directories
   - `mkdir source`
   - `mkdir build/`
     
3. You can follow the default wxOSX instructions at: https://github.com/wxWidgets/wxWidgets/blob/master/docs/osx/install.md
   ```
   mkdir build-cocoa-debug
   cd build-cocoa-debug
   ../configure --enable-debug
   make -j8
   ```
4. Or we can make some changes ourself, to change install directory and create static libraries
   ```
   ../configure --enable-debug --prefix=/Users/morbo/Documents/cpp_learning/libraries/wxwidgets --disable-shared
   make -j8
   make install
   ```
6. Now everything has been compiled successfully, we can start setting up wxwidgets to be used in Xcode 14, and hopefully other versions?
   - First cd to `cd /Users/morbo/Documents/cpp_learning/libraries/wxwidgets/bin` then run:
   - `./wx_config --cxxflags` and save the output, or atleast don't clear it in terminal
   - `./wx_config --libs` and save outout

8. Start up Xcode
9. Create a new commandline project
10. Feel free to compile the hello world immediately, it sets up the products folder in the sidebar. 
11. Once created the build settings of the project, and click on targets, ***not Projects***
12. Go to Build settings, search for **Architecture**
13. Under **Build Active Architecture Only** check yes if it's not already, make sure Debug and Release are also set.
14. Search for **Header Search Paths**
    - Add the directory of the headers, `/Users/yourusername/Documents/some/dir/libraries/wxwidgets/include/wx-3.2` set to **non-recursive**
15. Search for **Library Search Paths**
    - Add the path to the `/Users/yourusername/Documents/some/dir/libraries/wxwidgets/lib` directory under your install path, set to **recursive**
16. Search for **Other Linker Flags**
    - Add the premade list from the result of `wx_config --libs`, it will show up in one line, then seperate to seperate ones, it's fine.
17. Search for **Other C++**
    - Add immediately `-l/Users/yourusername/Documents/some/dir/libraries/wxwidgets/lib/wx/include/osx_cocoa-unicode-static-3.2/`
      - This above nonsense is completely and utterly needed in order to do ***anything*** at all in any IDE, it ***must** be in first place, otherwise it will completely fail!
    - Add the premake list from the result of `wx_config --cxxflags`, it will show in one line, it's fine. 
17. Search for `clang` or `llvm`, I use clang.
    - Under c++ language detect, set to c++17 std.
    - under C Languag detect, set to gnu11
18. Do whatever other crap you want.
19. Go to **Product -> Scheme -> Edit Scheme** Under Run, change to debug and uncheck release, currently the library is setup for debugging and not release, this will have to be repeated when changing to release.
20. Add the following code to your main.cpp from the hellowork tutorial https://docs.wxwidgets.org/latest/overview_helloworld.html , and see if it works:
```c++
//
//  main.cpp
// Start of wxWidgets "Hello World" Program
#include <wx/wx.h>
 
class MyApp : public wxApp
{
public:
    bool OnInit() override;
};
 
wxIMPLEMENT_APP(MyApp);
 
class MyFrame : public wxFrame
{
public:
    MyFrame();
 
private:
    void OnHello(wxCommandEvent& event);
    void OnExit(wxCommandEvent& event);
    void OnAbout(wxCommandEvent& event);
};
 
enum
{
    ID_Hello = 1
};
 
bool MyApp::OnInit()
{
    MyFrame *frame = new MyFrame();
    frame->Show(true);
    return true;
}
 
MyFrame::MyFrame()
    : wxFrame(nullptr, wxID_ANY, "Hello World")
{
    wxMenu *menuFile = new wxMenu;
    menuFile->Append(ID_Hello, "&Hello...\tCtrl-H",
                     "Help string shown in status bar for this menu item");
    menuFile->AppendSeparator();
    menuFile->Append(wxID_EXIT);
 
    wxMenu *menuHelp = new wxMenu;
    menuHelp->Append(wxID_ABOUT);
 
    wxMenuBar *menuBar = new wxMenuBar;
    menuBar->Append(menuFile, "&File");
    menuBar->Append(menuHelp, "&Help");
 
    SetMenuBar( menuBar );
 
    CreateStatusBar();
    SetStatusText("Welcome to wxWidgets!");
 
    Bind(wxEVT_MENU, &MyFrame::OnHello, this, ID_Hello);
    Bind(wxEVT_MENU, &MyFrame::OnAbout, this, wxID_ABOUT);
    Bind(wxEVT_MENU, &MyFrame::OnExit, this, wxID_EXIT);
}
 
void MyFrame::OnExit(wxCommandEvent& event)
{
    Close(true);
}
 
void MyFrame::OnAbout(wxCommandEvent& event)
{
    wxMessageBox("This is a wxWidgets Hello World example",
                 "About Hello World", wxOK | wxICON_INFORMATION);
}
 
void MyFrame::OnHello(wxCommandEvent& event)
{
    wxLogMessage("Hello world from wxWidgets!");
}
```
If you're lucky, it somehow magically works...exactly this code and probably not any others. But _technically_ you have compiled yourself some static opencv libraries and can pass your compiled program to any other mac arm64 user without fail.

Good luck!
     
