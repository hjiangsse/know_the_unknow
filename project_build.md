# cmake
## 1.1 A basic example:
### 1.1.1 Prepare your source code file directory:
.
├── CMakeLists.txt
├── TutorialConfig.h.in
└── tutorial.cxx

TutotialConfig.h.in:
```
// the configured options and settings for Tutorial
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

tutorial.cxx:
``` c++
#include <iostream>
#include "TutorialConfig.h"

using namespace std;

int main(int argc, char **argv) {
   if (argc < 2) {
       // report version
       std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "." << Tutorial_VERSION_MINOR << std::endl;
       std::cout << "Usage: " << argv[0] << " number" << std::endl;
   } else {
       const double inputValue = std::stod(argv[1]);
       cout << "Input value is " << inputValue << endl;
   }
   return 0;
}
```
CMakeLists.txt:
```
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.2)

# add config header file
configure_file(TutorialConfig.h.in TutorialConfig.h)
# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# add the executable
add_executable(Tutorial tutorial.cxx)

# add binary directory as a include directory
# so tutorial.cxx can use TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}")
```

when you cmake the project, _configure_file(TutorialConfig.h.in TutorialConfig.h)_ will generate a _TutorialConfig.h_
in the build(binary) directory,  _target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")_ will add
binary_dir as include directory, so we can include it in our main program.


### 1.1.2 Cmake and build the project:
put the previous 3 files in a directory called Step1, the make a new directory called Step1_build:
.
├── Step1
└── Step1_build

``` bash
cd Step1_build
cmake ../Step1
cmake --build .
```


## 1.2 Add library to project:
