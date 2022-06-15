# Serial Communication Library

[![test](https://github.com/sdustio/serial/actions/workflows/test.yml/badge.svg)](https://github.com/sdustio/serial/actions/workflows/test.yml)
[![build](https://github.com/sdustio/serial/actions/workflows/build.yml/badge.svg)](https://github.com/sdustio/serial/actions/workflows/build.yml)


This is a cross-platform library for interfacing with rs-232 serial like ports written in C++. It provides a modern C++ interface with a workflow designed to look and feel like PySerial, but with the speed and control provided by C++.

Serial is a class that provides the basic interface common to serial libraries (open, close, read, write, etc..) and requires no extra dependencies. It also provides tight control over timeouts and control over handshaking lines.

## Documentation

Website: http://wjwwood.github.io/serial/

API Documentation: http://wjwwood.github.io/serial/doc/1.1.0/index.html

## Install

**Dependencies**

Required:
* [cmake](http://www.cmake.org) - buildsystem

Optional (for documentation):
* [Doxygen](http://www.doxygen.org/) - Documentation generation tool
* [graphviz](http://www.graphviz.org/) - Graph visualization software

**Build and Install**

Get the code:

    git clone https://github.com/sdustio/serial.git

Build and Install:

    cmake -DCMAKE_INSTALL_PREFIX:STRING=$HOME/.local -H$(pwd) -B$(pwd)/build-release
    cmake --build $(pwd)/build-release --target install

Build the documentation:

    doxygen doc/Doxyfile


## Contribute

1. Change some code

2. Test

    cmake -DSERIAL_BUILD_TESTS:BOOL=TRUE -DCMAKE_BUILD_TYPE:STRING=Debug -H$(pwd) -B$(pwd)/build
    cmake --build $(pwd)/build --config Debug
    cd $(pwd)/build
    ctest
    ctest -T memcheck

3. Create [pull request](https://github.com/sdustio/serial/pulls) on github


### Authors

Michael Ding <yandy.ding@gmail.com>
William Woodall <wjwwood@gmail.com>
John Harrison <ash.gti@gmail.com>

### Contact

Michael Ding <yandy.ding@gmail.com>
