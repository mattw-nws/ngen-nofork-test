cmake_minimum_required(VERSION 3.10)
add_library(geopackage INTERFACE)
add_library(NGen::geopackage ALIAS geopackage)
target_include_directories(geopackage INTERFACE ${PROJECT_SOURCE_DIR}/include/geopackage)

find_package(Boost)
target_link_libraries(geopackage INTERFACE NGen::geojson Boost::boost)