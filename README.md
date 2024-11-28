# A minimal TT-Metal CMake template


This is a minimal working cmake template to start off your TT-Metal projects. It is essentially ~~a stripped down of~~ TT-Metal's CMakeLists and dependencies allowing your projects to easily be built allowing newbies to have a good easy-access on creating their kernels rather than trying to figure out what to folders/libs to import in their cmake files.

## Does this work?
This depends on how old this repo was last updated, TT-Metal's way of building may change so the more later this has been updated the more likely it is outdated but I will try my best to make it as up to date as possible. This template also assumes you're running on the near-latest commit of Metallium.


## Using this in your own project
If you're using this for your project, rename `my_foo` to your project name and add in your .cpp files into the executable target.

Example:
```cmake
add_executable(my_foo example.cpp)

to

add_executable(example_project_name 
    src/i_love_tt.cpp
    src/i_also_love_tt.cpp
)
```

```cmake
target_compile_definitions(my_foo PRIVATE
    ARCH_$<UPPER_CASE:$ENV{ARCH_NAME}>
    FMT_HEADER_ONLY
)

to

target_compile_definitions(example_project_name PRIVATE
    ARCH_$<UPPER_CASE:$ENV{ARCH_NAME}>
    FMT_HEADER_ONLY
)
```

## Setup
First, you have [built tt-metal first](https://github.com/tenstorrent/tt-metal/blob/main/INSTALLING.md).

After you've built tt-metal, check that you have set your `TT_METAL_HOME` and `ARCH_NAME` environment variables are correctly set. `TT_METAL_HOME` should be the repository that you've built TT-Metal on while `ARCH_NAME` is the name of your TT device.

## Building

Make sure you specify your CXX and C compiler for cmake to use. Additionally, set the `ENABLE_LIBCXX` flag on especially when using clang-17 for compiling.

```sh
# Clang-17
cmake -S . -B build -DCMAKE_CXX_COMPILER=clang++-17 -DCMAKE_C_COMPILER=clang-17 -DENABLE_LIBCXX=on

# GCC
cmake -S . -B build -DCMAKE_CXX_COMPILER=g++ -DCMAKE_C_COMPILER=gcc -DENABLE_LIBCXX=on
```

Next, run `make` to compile:

```sh
make -C build
```

## Running your projects

Finally, you can now run your Metallium project!

```sh
./build/my_foo
```

If your kernels fail to compile because they can't be found then check then path you've set it on. This example assumes you're executing it in the parent directory meaning it will find `./kernels/` folder for where the kernels are located.


### Build types
By default, this compiles in Release build time, which passes -O3 optimization flag. You can set `-DCMAKE_BUILD_TYPE` to `Debug` for no optimizing + debug or `RelWithDebInfo` with optimizing with debugging mode on.
