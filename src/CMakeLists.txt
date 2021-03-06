# The MIT License (MIT)
#
# Copyright (c) 2018 Mateusz Pusz
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

cmake_minimum_required(VERSION 3.12)

project(mp-units
    VERSION 0.6.0
    LANGUAGES CXX
)

# set path to custom cmake modules
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../cmake")

# include common tools and workarounds
include(common/cmake/scripts)

# use Conan configuration if available
conan_init(cmake)

# library definition
add_library(units INTERFACE)
#target_sources(units INTERFACE
#    include/units/dimension.h
#    include/units/quantity.h
#    include/units/unit.h
#
#    include/units/bits/tools.h
#    include/units/bits/type_list.h
#
#    include/units/si/base_dimensions.h
#    include/units/si/frequency.h
#    include/units/si/length.h
#    include/units/si/time.h
#    include/units/si/speed.h
#)
target_compile_features(units INTERFACE cxx_std_20)
target_link_libraries(units
    INTERFACE
        $<IF:$<TARGET_EXISTS:CONAN_PKG::fmt>,CONAN_PKG::fmt,fmt::fmt>
)
target_include_directories(units
    INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
)
if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    target_link_libraries(units
        INTERFACE
            $<IF:$<TARGET_EXISTS:CONAN_PKG::range-v3>,CONAN_PKG::range-v3,range-v3::range-v3>
    )
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    target_compile_options(units
    INTERFACE
        -Wno-literal-suffix
        -Wno-non-template-friend
    )

    if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 10.0)
        target_compile_options(units
            INTERFACE
                -fconcepts
        )
        target_link_libraries(units
            INTERFACE
                $<IF:$<TARGET_EXISTS:CONAN_PKG::range-v3>,CONAN_PKG::range-v3,range-v3::range-v3>
        )
    endif()
endif()
add_library(mp::units ALIAS units)

# installation info
install_targets(units)
install(DIRECTORY include/units
    DESTINATION include
    COMPONENT Devel
)

# generate configuration files and install the package
configure_and_install(../cmake/common/cmake/simple-config.cmake.in mp SameMajorVersion)
