cmake_minimum_required(VERSION 3.21)

include(FetchContent)
FetchContent_Declare(
        serious-cmake-library
        GIT_REPOSITORY "git@github.com:UsatiyNyan/serious-cmake-library.git"
        GIT_TAG main)
FetchContent_MakeAvailable(serious-cmake-library)

sl_project_prologue(CXX_STANDARD 20)
project(
        serious-libraries
        VERSION 0.0.2
        DESCRIPTION "collection of serious libraries"
        HOMEPAGE_URL "TODO"
        LANGUAGES CXX C)

cpmaddpackage(
        NAME serious-meta-library
        GIT_REPOSITORY "git@github.com:UsatiyNyan/serious-meta-library.git"
        GIT_TAG main)
cpmaddpackage(
        NAME serious-execution-library
        GIT_REPOSITORY "git@github.com:UsatiyNyan/serious-execution-library.git"
        GIT_TAG main)
cpmaddpackage(
        NAME serious-calculation-library
        GIT_REPOSITORY "git@github.com:UsatiyNyan/serious-calculation-library.git"
        GIT_TAG main)
cpmaddpackage(
        NAME serious-graphics-library
        GIT_REPOSITORY "git@github.com:UsatiyNyan/serious-graphics-library.git"
        GIT_TAG main)
cpmaddpackage(
        NAME serious-io-library
        GIT_REPOSITORY "git@github.com:UsatiyNyan/serious-io-library.git"
        GIT_TAG main)
cpmaddpackage(
        NAME serious-game-library
        GIT_REPOSITORY "git@github.com:UsatiyNyan/serious-game-library.git"
        GIT_TAG main)
