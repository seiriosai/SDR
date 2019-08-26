cmake_minimum_required(VERSION 3.8)
project(fdr)

# set(CMAKE_BUILD_TYPE "Release")
# set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
# set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")

find_package(OpenCV REQUIRED)
find_package(Eigen3 REQUIRED)

set(CMAKE_CXX_STANDARD 11)

include_directories(${OpenCV_INCLUDE_DIRS} ${EIGEN3_INCLUDE_DIRS})

list(APPEND SDR_SOURCE_FILES 
    FastDR.cpp 
    FastDR.h 
    SurfaceFitting.cpp 
    SurfaceFitting.h 
    ArgsParser.h 
    Evaluator.h 
    Parameters.h 
    TimeStamper.h 
    Utilities.hpp
)

add_library(sdr ${SDR_SOURCE_FILES})
target_link_libraries(sdr PUBLIC GCoptimization)

add_executable(fdr main.cpp)
target_link_libraries(fdr ${OpenCV_LIBS} Eigen3::Eigen sdr)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
set(cmake_package_name sdr)
set(targets_export_name ${cmake_package_name}Targets CACHE INTERNAL "")

install(TARGETS sdr EXPORT ${targets_export_name} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(TARGETS GCoptimization EXPORT ${targets_export_name} ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})

install(FILES
    FastDR.h 
    SurfaceFitting.h 
    ArgsParser.h 
    Evaluator.h 
    Parameters.h 
    TimeStamper.h 
    DESTINATION include
)

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated" CACHE INTERNAL "")
set(cmake_files_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${cmake_package_name}")
set(version_file "${generated_dir}/${cmake_package_name}ConfigVersion.cmake")
write_basic_package_version_file(${version_file} VERSION 1.0.12 COMPATIBILITY AnyNewerVersion)

install(EXPORT ${targets_export_name} DESTINATION ${cmake_files_install_dir})

set(config_file "${generated_dir}/${cmake_package_name}Config.cmake")
configure_package_config_file("Config.cmake.in"
  "${config_file}" INSTALL_DESTINATION ${cmake_files_install_dir})
install(FILES ${version_file} ${config_file}
  DESTINATION ${cmake_files_install_dir})