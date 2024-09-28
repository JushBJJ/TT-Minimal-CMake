# A minimal TT-Metal CMake template

This is a minimal working cmake template to start off your TT-Metal projects. It is essentially a stripped down of TT-Metal's CMakeLists and depenencies allowing your projects to easily be built allowing newbies to have a good easy-access on creating their kernels rather than trying to figure out what to folders/libs to import in their cmake files.

## Instructions

1. Copy CMakeLists.txt and cmake/ into your project's directory
2. Edit CMakeLists.txt to include your .cpp files and name your project name

## Running your projects

1. `mkdir -p build/`
2. `cmake ..`
3. `make`
4. Run your executable. \*

\* You may need to run the executable in the parent directory so TT-Metal can find your kernels.
