# Introduction

Wayland is an object oriented display protocol, which features request
and events. Requests can be seen as method calls on certain objects,
whereas events can be seen as signals of an object. This makes the
Wayland protocol a perfect candidate for a C++ binding.

The goal of this library is to create such a C++ binding for Wayland
using the most modern C++ technology currently available, providing
an easy to use C++ API to Wayland.

# Requirements

To build this library, a recent version of cmake is required. Furthermore,
a recent C++ Compiler with C++11 support, such as GCC or clang, is required.
Also, pugixml is required to build the XML protocol scanner. Apart from the
Wayland libraries, there are no further library dependencies.

The documentation is autogenerated using Doxygen, therefore doxygen as
well as graphviz is required.

# Building

## Library

To build the library, `cmake ..` needs to executed in a newly created
`build` directory in the root directory of the repository, followed
by a `make`. After that, `make install` will install the library.

There are several CMake variables that can be set in order to
customise the build and install process:

CMake Variable              | Effect
--------------------------- | ------
`CMAKE_CXX_COMPILER`        | C++ compiler to use
`CMAKE_CXX_FLAGS`           | Additional flags for the C++ compiler
`CMAKE_INSTALL_PREFIX`      | Prefix folder, under which everything is installed
`CMAKE_INSTALL_LIBDIR`      | Library folder relative to the prefix
`CMAKE_INSTALL_INCLUDEDIR`  | Header folder relative to the prefix
`CMAKE_INSTALL_BINDIR`      | Binary folder relative to the prefix
`CMAKE_INSTALL_DATAROOTDIR` | Shared folder relative to the prefix
`CMAKE_INSTALL_DOCDIR`      | Dcoumentation folder relative to the prefix
`CMAKE_INSTALL_MANDIR`      | Manpage folder relative to the prefix
`BUILD_SCANNER`             | Whether to build the scanner
`BUILD_LIBRARIES`           | Whether to build the libraries
`BUILD_DOCUMENTATION`       | Whether to build the documentation
`BUILD_EXAMPLES`            | Whether to build the examples

The installation root can also be changed using the environment variable
`DESTDIR` when using `make install`.

## Documentation

If the requirements are met, the documentation will normally be built
automatically. HTML pages, LaTeX source files as well as manpages are generated.

To build the documentation manually, `doxygen` needs to be executed
in the root directory of the repository. The resulting documentation
can then be found in the `doc` directory. The required Doxyfile is
available after the library has been built. The documentaion is also
online availabe [here](https://nilsbrause.github.io/waylandpp_docs/).

## Example programs

To build the example programs the `BUILD_EXAMPLES` option needs to be enabled
during the build. The resulting binaries will be put under the `example`
directory inside the build directory. They can be run directly without
installing the library first.

To build the example programs manually, `make` can executed in
the example directory after the library has been built and installed.

# Usage

In the following, it is assumed that the reader is familiar with
basic Wayland concepts and at least version 11 of the C++
programming language.

Each interface is represented by a class. E.g. the `wl_registry`
interface is represented by the `registry_t` class.

An instance of a class is a wrapper for a Wayland object (a `wl_proxy`
pointer). If a copy is made of a particualr instance, both instances
refer to the same Wayland object. The underlying Wayland object is
destroyed once there are no copies of this object left. Only a few
classes are non-copyable, namely `display_t` and `egl_window_t`.
There are also special rules for proxy wrappers and the use of
foreigen objects. Refer to the documentation for more details.

A request to an object of a specific interface corresponds to a method
in this class. E.g. to marshal the `create_pool` request on an
`wl_shm` interface, the `create_pool()` method of an instance of
`shm_t` has to be called:

    shm_t shm;
    int fd;
    int32_t size;
    // ... insert the initialisation of the above here ...
    shm_pool_t shm_pool = shm.create_pool(fd, size);

Some methods return newly created instances of other classes. In this
example an instance of the class `shm_pool_t` is returned.

Events are implemented using function objects. To react to an event, a
function object with the correct signature has to be assigned to
it. These can not only be static functions, but also member functions
or closures. E.g. to react to global events from the registry using a
lambda expression, one could write:

    registry.on_global() = [] (uint32_t name, std::string interface,
                               uint32_t version)
      { std::cout << interface << " v" << version << std::endl; };

An example for using member functions can be found in
example/opengles.cpp or example/shm.cpp.

The Wayland protocol uses arrays in some of its events and requests.
Since these arrays can have arbitrary content, they are not directly
mapped to a std::vector. Instead there is a new type array_t, which
can converted to and from a std::vectory with an user specified type.
For example:

    keyboard.on_enter() = [] (uint32_t serial, surface_t surface,
                              array_t keys)
      { std::vector<uint32_t> vec = keys; };

To compile code that using this library, pkg-config can be used to
take care of the compiler flags. Assuming the source file is called
`foo.cpp` and the executable shall be called `foo` type:

    $ c++ -c foo.cpp `pkg-config --cflags wayland-client++` -o foo.o
    $ c++ foo.o `pkg-config --libs wayland-client++` -o foo

If the library and headers are installed in the default search paths
of the compiler, the linker flag `-lwayland-client++` can also
directly be specified on the command line.

If the Wayland cursor classes and/or EGL is used, the corresponding
libreries `wayland-cursor++` and/or `wayland-egl++` need to be linked
in as well. If any extension protocols such as xdg-shell are used,
the library `wayland-client-extra++` should be linked in as well.

Further examples can be found in the examples/Makefile.
