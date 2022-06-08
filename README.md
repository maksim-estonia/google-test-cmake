## Google Test: CMake

- [Google Test: CMake](#google-test-cmake)
  - [Quickstart: Building with CMake](#quickstart-building-with-cmake)
  - [Samples](#samples)
  - [CMake Tools](#cmake-tools)

### Quickstart: Building with CMake

**Set up a project**

CMake uses a file name `CMakeLists.txt` to configure the build system for a project. You'll use this file to set up your project and declare a dependency on GoogleTest.

First create a directory for your project.

```
mkdir my_project
cd my_project
```

Next, you'll create the `CMakeLists.txt` file and declare a dependency on GoogleTest. There are many ways to express dependencies in the CMake ecosystem; here, we'll use the `FetchContent` Cmake module. To do this, in your project directory, create a file named `CMakeLists.txt` with the following contents:

```cmake
cmake_minimum_required(VERSION 3.14)
project(my_project)

# GoogleTest requires at least C++11
set(CMAKE_CXX_STANDARD 11)

include(FetchContent)
FetchContent_Declare(
    googletest
    url https://github.com/google/googletest/archive/609281088cfefc76f9d0ce82e1ff6c30cc3591e5.zip
)

# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)
```

For more information about how to create `CMakeLists.txt` files, see the [CMake tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)

**Create and run a binary**

With GoogleTest declared as a dependency, you can use GoogleTest code within your project.

As an example, create a file named `hello_test.cpp` in your project directory with the following contents:

```cpp
#include <gtest/gtest.h>

// Demonstrate some basic assertions.
TEST(HelloTest, BasicAssertions) {
  // Expect two strings not to be equal.
  EXPECT_STRNE("hello", "world");
  // Expect equality.
  EXPECT_EQ(7 * 6, 42);
}
```

GoogleTest provides assertions that you can use to test the behaviour of your code. The above example includes the main GoogleTest header file and demonstrates some basic [assertions](https://google.github.io/googletest/reference/assertions.html). 

To build the code, add the following to the end of your `CMakeLists.txt` file:

```cmake
enable_testing()

add_executable(
  hello_test
  hello_test.cc
)
target_link_libraries(
  hello_test
  gtest_main
)

include(GoogleTest)
gtest_discover_tests(hello_test)
```

The above configuration enables testing in CMake, declares the C++ test binary you want to build (`hello_test`), and links to GoogleTest (`gtest_main`). The last two lines enable CMake's test runner to discover the tests included in the binary, using the GoogleTest CMake module.

Now you can build and run your test:

```
cmake -S . -B build
cmake --build build
cd build
ctest
```

![cmake-build-test](images/cmake-build-test.png)

### Samples

- [sample-1](my_samples/sample-1/sample-1.md): shows the basic steps of using googletest to test C++ functions
- [sample-2](my_samples/sample-2/sample-2.md): shows a more complex unit test for a class with multiple member functions
- [sample-3](my_samples/sample-3/sample-3.md): uses a test fixture 
- [sample-4](my_samples/sample-4/sample-4.md): teaches you how to use googletest and `googletest.h` together to get the best of both libraries
- [sample-5](my_samples/sample-5/sample-5.md): puts shared testing logic in a base test fixture, and reuses it in derived fixtures
- [sample-6](my_samples/sample-6/sample-6.md): demonstrates type-parametrized tests
- [sample-7](my_samples/sample-7/sample-7.md): teaches the basics of value-parametrized tests
- [sample-8](my_samples/sample-8/sample-8.md): shows using `Combine()` in value-parametrized tests
- [sample-9](my_samples/sample-9/sample-9.md): shows use of listener API to modify Google Test's console output and the use of its reflection API to inspect test results
- [sample-10](my_samples/sample-10/sample-10.md): shows use of the listener API to implement a primitive memory leak checker

### CMake Tools

[source](https://code.visualstudio.com/docs/cpp/CMake-linux)

The CMake Tools extension integrates VSCode and CMake to make it easy to configure, build, and debug your C++ project.

**Ensure the CMake is installed**

```
cmake --version
```

**Ensure that development tools are installed**

```
gcc -v
sudo apt-get install build-essential gdb
```

**Create a CMake project**

```
mkdir cmakeQuickStart
cd cmakeQuickStart
code .
```

The `code .` command opens VSCode in the current directory, which becomes your "workspace".

Open the command palette and run the `CMake: Quick Start` command.

Enter a project name, this will be written to `CMakeLists.txt` and a few initial source files. 

Next, select `Executable` as the project type to create a basic source file (`main.cpp`) that includes a basic `main()` function.

Note: if you had wanted to create a basic source and header file, you would have selected `Library` instead.

**Build**

Open the command palette and run the `Cmake: Build` command, or select the `Build` button from the status bar.

**Debug**

Put a breakpoint on a line, then open the command palette and run `Cmake: Debug`.

**Run tests**

Open the command palette and run the `CMake: Run Tests` command. 

Manually (after builing):

```
cd build
ctest
```

[Additional information](https://github.com/microsoft/vscode-cmake-tools/blob/main/docs/README.md)



