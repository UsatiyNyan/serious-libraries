# serious-libraries

For serious programmers.

## serious project structure

This is a collection of "serious libraries".

The only library included explicitly through FetchContent is a "serious cmake library".
This is done to delegate all package management and build work to this library,
avoiding unnecessary dependencies in root project.
Also, "serious cmake library" is intended to be backwards-compatible and always up-to-date.

Every other "serious library" is installed as a package (currently using CPM) and are unversioned.

## collection of serious libraries in graph-sorted dependency order

- https://github.com/usatiynyan/serious-cmake-library
- https://github.com/UsatiyNyan/serious-meta-library
- https://github.com/UsatiyNyan/serious-calculation-library
- https://github.com/UsatiyNyan/serious-graphics-library
- https://github.com/UsatiyNyan/serious-serialization-library

## serious project roadmap

There is none currently.

Whatever ends up here is a library that is a distilled version of my experience with a particular technology.

## articles

- [CMake is all you need](articles/CMake%20is%20all%20you%20need.md)

