cmake_minimum_required(VERSION 3.21)

include(FetchContent)
FetchContent_Declare(
        serious-cmake-library
        GIT_REPOSITORY git@github.com:UsatiyNyan/serious-cmake-library.git
        GIT_TAG main)
FetchContent_MakeAvailable(serious-cmake-library)

sl_project_prologue(CXX_STANDARD 20)
project(
        serious-libraries
        VERSION 0.0.1
        DESCRIPTION "collection of serious libraries"
        HOMEPAGE_URL "TODO"
        LANGUAGES CXX C)
