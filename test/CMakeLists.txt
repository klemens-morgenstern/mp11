# Copyright 2018 Peter Dimov
# Distributed under the Boost Software License, Version 1.0.
# See accompanying file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt

include(BoostTestJamfile OPTIONAL RESULT_VARIABLE HAVE_BOOST_TEST)

if(HAVE_BOOST_TEST)

boost_test_jamfile(FILE Jamfile LINK_LIBRARIES Boost::mp11 Boost::core)
boost_test(SOURCES check_cmake_version.cpp ARGUMENTS ${PROJECT_VERSION} LINK_LIBRARIES Boost::core Boost::config)

endif()

if(CMAKE_VERSION VERSION_GREATER_EQUAL "3.28.0")
    add_library(boost_mp11_mod_module_test module.cpp)
    target_link_libraries(boost_mp11_mod_module_test PUBLIC boost_mp11_mod_module)
    target_compile_features(boost_mp11_mod_module_test PUBLIC cxx_std_20)
endif()