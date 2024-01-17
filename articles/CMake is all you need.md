# The wast-wast world of modern C++

Modern C++ has many out-of-the-box features, and many more can be enabled via external tools and libraries.
Even though existence of most features makes sense, the sheer amount of side effects and quirky interactions,
we as programmers have to keep up with, makes our job overwhelming.

It goes without saying that tons of libraries, language features and design decisions are frequently misused,
leading to unmanageable and overly complicated code. Which is why C++ gets a bad reputation.

To conquer complexity and avoid brain segfaulting, we, developers, should establish reasonable limitations.
We must strive for simplicity and minimalism.

Achieving, at least, relative simplicity in our code bases is, ironically, not a simple task,
so let’s start with the build tools, dependencies and code delivery. 

# Package management

One of the modern go-to ways of handling dependencies and delivering code is to use a package manager.
In the case of C++ there are lots of them: vcpkg/conan/build2/… and unlike in other modern languages (e.g. Go or Rust)
the **standard** package manager is up for discussion. 

As much as package managers promise to make your life easier, their use comes with major inconveniences.
First of all, adding additional dependencies and files to your repository imposes maintenance work to conform to their
infrastructure or (if you decide to) in case of migrating to another package manager. 
Second, for the most part, they are already using CMake to build your project. 
Which means that you are not completely free from typical problems of dependency management,
though these problems become more obscure due to a "level of abstraction".
So what’s the point of even using a package manager?
And third, what if your go-to package manager doesn’t have the dependency you want?
Do you stop everything, fork existing repository, do the maintenance work and publish it?
Or do you stop and rethink your life decisions?

At the time when I was working in the robotics field we had to package everything ourselves.
Not only did the core framework ([ROS2](https://docs.ros.org/)) work well with deb packages,
but also our whole environment was already containerized. 
Working inside of docker meant working as close to prod as possible.
That also meant that we had to package our dependencies.
We had to think twice before adding extra dependencies, and even though it was quite time-consuming
and sometimes frustrating, at least project structure was consistent.

# Pure and Simple*

There’s on the other hand a contrarian way of handling dependencies, building and delivering code,
which is none. Literally.

Look for example at the collection of universally loved [stb libraries](https://github.com/nothings/stb),
[ImGui](https://github.com/ocornut/imgui), [miniaudio](https://github.com/mackron/miniaudio) and so on.
These are purely self-contained **and** highly configurable libraries.
They don’t impose any restrictions on how to use them. Just add sources to your program and voila!
Something neat at your disposal!

Even though these libraries don’t restrict you, on how to use them, they are not allowed to have dependencies,
which leads to wheels being reinvented, every time they have to do something trivial.
Simply because they have to be self-contained and self-sufficient.

I love using these libraries, but I wouldn’t personally write code like that, call it a skill issue,
but my feeble mind wouldn’t be able to work in a single header/source file.

There is also an interesting take on the purism of build systems:
Tsoding's [C build tool written in C that uses purely C compiler to build C projects](https://github.com/tsoding/nobuild).

# Going platform specific

I would not make a strong case against Make, MSBuild, Ninja, XCode, and other platform-specific build tools.
If your tool is right for the environment you are in, then there is no reason to CMake your own life harder,
to try and force crossplatformness upon your project.

From my personal experience, MSBuild is probably the worst when it comes to dependency management,
especially when dependencies are already using CMake. I witnessed with my own eyes the "crimes"
of a desperate windows DevOps, trying to get sh*t done, hardcoding every internal CMake variable to
some value to produce a DLL that doesn’t even work half the time, oh boy!

Probably, if your project is generic enough, it wouldn’t hurt to eventually switch to CMake,
because one day a guy like me could come in and deeply appreciate your effort!
And maybe even contribute!

# CMake is all you need

When I say this, I really mean it.

## dependencies

FetchContent plus a small .cmake script can make any repository into a dependency for your project.
I am not going to explain completely, how to use it, but as an example, using
[fmt library](https://github.com/fmtlib/fmt) is as easy as that:
```cmake
FetchContent_Declare(
        NAME fmt
        GIT_REPOSITORY "git@github.com:fmtlib/fmt.git"
        GIT_TAG 10.1.1)

# this line is what includes top level CMakeLists.txt of fmt
FetchContent_MakeAvailable(fmt) 

# don't forget to link against the library!
target_link_libraries(... fmt::fmt)
```

There’s a neat wrapper around FetchContent functionality - CMake Package Manager(CPM),
which makes writing the same configuration even easier!
```cmake
# NOTE: CPMAddPackage automatically includes top level CMakeLists.txt of the project
CPMAddPackage(
        NAME fmt
        GIT_REPOSITORY "git@github.com:fmtlib/fmt.git"
        GIT_TAG 10.1.1)

# don't forget to link against the library!
target_link_libraries(... fmt::fmt)
```

Using only CMake to manage dependencies, whether by FetchContent or CPM has following advantages:
- An easier project configuration, you just call `cmake`,
and more or less understand how it will setup project and its dependencies;
- Configuration becomes much more reproducible since it is self-contained.

And even though CPM is called a package manager, by virtue of existing on the same level as the rest of your project configuration, it doesn’t have the same disadvantages:
- It doesn’t limit downstream projects from using any other environment, since it will set up itself if needed,
and will install all required dependencies;
- It doesn’t require non-CMake files in your project to work correctly;
- It doesn’t require any binaries to be installed on the client’s machine, other than CMake itself.

Whilst you could do a similar setup for package managers discussed earlier, with custom targets,
I would argue package manager itself in this case is redundant.

One thing to consider, is that FetchContent can be relatively slow, but CPM provides optional caching of dependencies.
In my experience caching usually isn’t even needed if you don’t try to download the entirety of Boost.

Important to note, that not every dependency is made equal, not every CMakeLists.txt is written with the thought
of being not the “top level” project in mind, leading to leaky configurations.
This results in many unwanted test/example targets cluttering your project.
But it all is easily fixable with a couple of lines of CMake code.
If you are interested, you can find examples of how to resolve different kinds of dependencies
[here](https://github.com/UsatiyNyan/serious-graphics-library/tree/main/dependencies) and
[here](https://github.com/UsatiyNyan/serious-music-visualizer/tree/main/dependencies).

And what do we get if we handle dependencies correctly and apply best practices of writing CMake configuration?
An easily pluggable dependency! Yes, just like that, your code, if you decide to publish it,
can easily be reused employing the same method. Isn’t that just so neat?

## and much more

Since there are obvious problems with C++ itself, that sometimes restricts our ability to write the desirable code,
that for example, supports compile-time reflection (which should come in some future standard, but God knows when
will it come to compilers), library implementers often rely on code generation.

And what do they use to generate code? Some other scripting language? A separate code generation tool?
Or compiler-specific features, from Clang for example? All of these approaches either assume the environment of the user
or impose limitations on said environment. Could these approaches be substituted by a CMake script?
Probably some of them.

As an example there is an interesting library, [Boost-PFR](https://github.com/boostorg/pfr), that “enables” introspection
through a simple concept - “tying structs(aggregates) as tuples”. Though I disagree with some decisions of that library,
the most sticking out part of it, for me, is using Python script to generate code that enables “tying of structs”. 
Also, supplying `_generated.cpp` code as a part of the library smells like a bad practice.
Could this be just a CMake script?
Yes it could, see code [here](https://github.com/UsatiyNyan/serious-meta-library/blob/main/cmake/generate_tie_as_tuple.cmake)
and tests [here](https://github.com/UsatiyNyan/serious-meta-library/blob/main/test/src/tuple_test.cpp#L37).

Though this example is simple enough to be substituted with just the CMake script, you probably could come up
with something much more complex that requires right tool for the job (like Protobuf), the dependency on that tool
should either be packaged as an explicit dependency of your project, 
or correctly asserted and required during the configuration stage.

# Lessons Learned

At the beginning of my programming journey, one of the biggest roadblocks was not even in the code itself.
It was in the environment and dependency setup. And I don’t think that I am alone in that.
Even though learning about Docker at the time was a godsent, since I could at least “freeze” my configuration
when it started working, it limited the project's potential to work on different platforms,
to work in user space and not from a container.
So, after walking this painful path, I’ve came to the conclusion,
that all I really needed was CMake and a set of best practices.

The true lesson here is not even about bringing CMake to its full potential, but rather about mindful consumption
of whatever new shiny thing you want to add to your project.
When contemplating adding an extra dependency, questions you should be asking yourself are:
- Can I reach the same results using my current tools?
- How much work would cost me to maintain new technology?
- Would it be clear, why new technology is added, to other developers?
- Would my project stay reusable in the future?

Exactly this mindset, in my opinion, leads towards a simpler and more consistent project structure.

# Further exploration 

CMake is a DSL. And the domain of that language is the environment in which you are building and delivering your project.
By learning CMake you would understand how that environment works. And it actually doesn’t take that much time to learn.
The only enemy might be in your head, CMake isn’t that, CMake is your friend.

One of the best starting points for me was this [gitbook](https://cliutils.gitlab.io/modern-cmake/).

As a reliable source of best practices and good examples, I highly recommend looking at the
[cpp-best-practices repo](https://github.com/cpp-best-practices/cppbestpractices) and watching through some videos
that support its contents (for example about [CPM](https://www.youtube.com/watch?v=dZMU3iAPhtI)).
And if you want more concrete examples, you could find them in a collection of [serious libraries](https://github.com/UsatiyNyan/serious-libraries).

Thank you for reading. Email me @ us4tiyny4n@gmail.com.

