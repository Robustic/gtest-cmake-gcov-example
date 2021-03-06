cmake_minimum_required(VERSION 2.8)

# Options. Turn on with 'cmake -Dmyvarname=ON'.
option(test "Build all tests." ON) # Makes boolean 'test' available.

# Make PROJECT_SOURCE_DIR, PROJECT_BINARY_DIR, and PROJECT_NAME available.
set(PROJECT_NAME MyProject)
project(${PROJECT_NAME})

set(CMAKE_CXX_FLAGS "-g -Wall")
set(CMAKE_C_FLAGS "-g -Wall")

# force static runtime libraries for msvc builds
if(MSVC)
  set(variables 
    CMAKE_CXX_FLAGS_DEBUG
    CMAKE_CXX_FLAGS_RELEASE
	CMAKE_CXX_FLAGS_RELWITHDEBINFO
	CMAKE_CXX_FLAGS_MINSIZEREL
  )
  foreach(variable ${variables})
	if(${variable} MATCHES "/MD")
	  string(REGEX REPLACE "/MD" "/MT" ${variable} "${${variable}}")
	endif()
  endforeach()
endif()

# If you want your own include/ directory, set this, and then you can do
# include_directories(${COMMON_INCLUDES}) in other CMakeLists.txt files.
# set(COMMON_INCLUDES ${PROJECT_SOURCE_DIR}/include)
################################
# Normal Libraries & Executables
################################
add_library(libbar 
    libbar/bar.cpp
    libbar/bar.h
    )
add_library(libfoo 
    libfoo/foo.cpp
    libfoo/foo.h
    )
add_executable( main_noTest
    main/main.cpp
    )
# Key idea: SEPARATE OUT your main() function into its own file so it can be its
# own executable. Separating out main() means you can add this library to be
# used elsewhere.
include_directories( 
    libfoo/
    libbar/
    )

target_link_libraries( main_noTest 
    libbar 
    libfoo
    )

################################
# Testing
################################
if (test)
  if (APPLE)
    add_definitions(-DGTEST_USE_OWN_TR1_TUPLE)
    add_definitions(-D__GLIBCXX__)
  endif (APPLE)

  # This adds another subdirectory, which has 'project(gtest)'.
  #add_subdirectory(lib/gtest-1.6.0)
  add_subdirectory(gtest/googletest)
  #add_subdirectory(test)

  enable_testing()

  # Include the gtest library. gtest_SOURCE_DIR is available due to
  # 'project(gtest)' above.
  include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})

  SET( GTEST_LIBRARIES gtest gtest_main gmock gmock_main )
  SET( CMAKE_BUILD_TYPE Debug )


  include( cmake/CodeCoverage.cmake )
  SET(CMAKE_CXX_FLAGS "-g -O0 --coverage -fprofile-arcs -ftest-coverage")
  SET(CMAKE_C_FLAGS "-g -O0 --coverage -fprofile-arcs -ftest-coverage")

  add_subdirectory( test )

  ###############
  ## Unit Tests
  ###############
  #add_executable(runUnitTests test_project1.cpp)

  ## Standard linking to gtest stuff.
  #target_link_libraries(runUnitTests gtest gtest_main)

  ## Extra linking for the project.
  #target_link_libraries(runUnitTests project1_lib)

  ## This is so you can do 'make test' to see all your tests run, instead of
  ## manually running the executable runUnitTests to see those specific tests.
  #add_test(NAME that-test-I-made COMMAND runUnitTests)

  ## You can also omit NAME and COMMAND. The second argument could be some other
  ## test executable.
  #add_test(that-other-test-I-made runUnitTests)
endif()


#enable_testing()

#OPTION( ENABLE_COVERAGE "Use gcov" ON )
#IF(ENABLE_COVERAGE)
#    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fprofile-arcs -ftest-coverage")
#    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fprofile-arcs -ftest-coverage")
#    SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fprofile-arcs -ftest-coverage")
#ENDIF()
#INCLUDE(CodeCoverage.cmake)
#SET(CMAKE_CXX_FLAGS "-g -O0 --coverage -fprofile-arcs -ftest-coverage")
#SET(CMAKE_C_FLAGS "-g -O0 --coverage -fprofile-arcs -ftest-coverage")
#
#SETUP_TARGET_FOR_COVERAGE(
#    my_coverage_target  # Name for custom target.
#    testfoo         # Name of the test driver executable that runs the tests.
#    # NOTE! This should always have a ZERO as exit code
#    # otherwise the coverage generation will not complete.
#    coverage            # Name of output directory.
#    )

#add_subdirectory(main)
#add_subdirectory(test)
#add_subdirectory(libfoo)
#add_subdirectory(libbar)
