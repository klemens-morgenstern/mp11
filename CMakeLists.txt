# Copyright 2018, 2019 Peter Dimov
# Distributed under the Boost Software License, Version 1.0.
# See accompanying file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt

cmake_minimum_required(VERSION 3.5...3.16)

project(boost_mp11 VERSION 1.88.0 LANGUAGES CXX)

add_library(boost_mp11 INTERFACE)
add_library(Boost::mp11 ALIAS boost_mp11)

target_include_directories(boost_mp11 INTERFACE include)
target_compile_features(boost_mp11 INTERFACE cxx_alias_templates cxx_variadic_templates cxx_decltype)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)

  # Fetch support files

  message(STATUS "Fetching BoostFetch.cmake")

  file(DOWNLOAD
    "https://raw.githubusercontent.com/boostorg/cmake/develop/include/BoostFetch.cmake"
    "${CMAKE_BINARY_DIR}/fetch_and_include/BoostFetch.cmake"
  )

  include("${CMAKE_BINARY_DIR}/fetch_and_include/BoostFetch.cmake")

  boost_fetch(boostorg/cmake TAG develop NO_ADD_SUBDIR)

  FetchContent_GetProperties(boostorg_cmake)

  list(APPEND CMAKE_MODULE_PATH ${boostorg_cmake_SOURCE_DIR}/include)

  # Testing (off by default)

  option(BUILD_TESTING "Build the tests." OFF)
  include(CTest)

  if(BUILD_TESTING)

    set(BUILD_TESTING OFF) # hide cache variable

    boost_fetch(boostorg/assert TAG develop EXCLUDE_FROM_ALL)
    boost_fetch(boostorg/config TAG develop EXCLUDE_FROM_ALL)
    boost_fetch(boostorg/core TAG develop EXCLUDE_FROM_ALL)
    boost_fetch(boostorg/static_assert TAG develop EXCLUDE_FROM_ALL)
    boost_fetch(boostorg/throw_exception TAG develop EXCLUDE_FROM_ALL)

    unset(BUILD_TESTING)

  endif()

  # Do not use the default BoostInstall versioned layout on Windows when standalone
  set(BOOST_INSTALL_LAYOUT "system" CACHE STRING "Installation layout (versioned, tagged, or system)")

  include(BoostInstall)

  boost_install(TARGETS boost_mp11 HEADER_DIRECTORY include/)

endif()


if(CMAKE_VERSION VERSION_GREATER_EQUAL "3.28.0")
    function(add_module name)
        add_library(boost_mp11_mod_${name})
        target_sources(boost_mp11_mod_${name} PUBLIC FILE_SET CXX_MODULES FILES module/${name}.cpp)
        target_include_directories(boost_mp11_mod_${name} PRIVATE include)
        target_compile_features(boost_mp11_mod_${name} PUBLIC cxx_std_20)
        target_compile_definitions(boost_mp11_mod_${name} PUBLIC BOOST_MP11_MODULE_EXPORT=export)
    endfunction()

    add_module(integral)
    add_module(list)
    target_link_libraries(boost_mp11_mod_list PUBLIC boost_mp11_mod_integral boost_mp11_mod_utility)
    add_module(utility)
    target_link_libraries(boost_mp11_mod_utility PUBLIC boost_mp11_mod_integral)
    add_module(algorithm)
    target_link_libraries(boost_mp11_mod_algorithm PUBLIC boost_mp11_mod_integral boost_mp11_mod_set boost_mp11_mod_utility boost_mp11_mod_function)
    add_module(set)
    target_link_libraries(boost_mp11_mod_algorithm PUBLIC boost_mp11_mod_list boost_mp11_mod_function)

    add_module(function)
    target_link_libraries(boost_mp11_mod_algorithm PUBLIC boost_mp11_mod_integral boost_mp11_mod_utility)
    add_module(bind)
    add_module(lambda)
    add_module(tuple)

    add_module(mp11)
    add_dependencies(
        boost_mp11_mod_mp11
        boost_mp11_mod_integral
        boost_mp11_mod_list
        boost_mp11_mod_utility
        boost_mp11_mod_algorithm
        boost_mp11_mod_set
        boost_mp11_mod_function
        boost_mp11_mod_bind
        boost_mp11_mod_lambda
        boost_mp11_mod_tuple)

endif()

if(BUILD_TESTING)

  add_subdirectory(test)

endif()
