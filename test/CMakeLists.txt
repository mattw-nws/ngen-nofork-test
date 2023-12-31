project(test)

include(CTest)
add_subdirectory(googletest)
include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})
include_directories(${gmock_SOURCE_DIR}/include ${gmock_SOURCE_DIR})
include_directories(${PROJ_ROOT_INCLUDE_DIR})

if (NGEN_ACTIVATE_PYTHON)
    include_directories(${PROJECT_SOURCE_DIR}/extern/pybind11/include)
    add_compile_definitions(NGEN_BMI_PY_TESTS_ACTIVE)
endif()

if(MPI_ACTIVE)
    add_compile_definitions(NGEN_MPI_TESTS_ACTIVE)
endif()

add_compile_definitions(NGEN_BMI_CPP_LIB_TESTS_ACTIVE)

if(BMI_C_LIB_ACTIVE)
    add_compile_definitions(NGEN_BMI_C_LIB_TESTS_ACTIVE)
endif()

if(BMI_FORTRAN_ACTIVE)
    add_compile_definitions(NGEN_BMI_FORTRAN_LIB_TESTS_ACTIVE)
endif()

if(LSTM_TORCH_LIB_ACTIVE)
    add_compile_definitions(LSTM_TORCH_LIB_TESTS_ACTIVE)
endif()

if(NGEN_ACTIVATE_ROUTING)
    add_compile_definitions(ROUTING_PYBIND_TESTS_ACTIVE)
endif()

if(NETCDF_ACTIVE)
    add_compile_definitions(NETCDF_ACTIVE)
endif()

# Add test source files (e.g., test1.cpp)
#add_executable(test_hymod models/hymod/include/HymodTest.cpp)
#target_link_libraries(test_hymod gtest gtest_main)

function(add_test TESTNAME NUM_FILES)
    math(EXPR NUM_LINKS "${ARGC} - ${NUM_FILES}")
    list(SUBLIST ARGN 0 ${NUM_FILES} SOURCE_FILES)
    list(SUBLIST ARGN ${NUM_FILES} ${NUM_LINKS} LINKED_LIBS)

    # Also link these testing libraries for anything
    # TODO: verify these do not remove current value at i = 0, but instead push it forward
    list(INSERT LINKED_LIBS 0 gtest gtest_main)

    # Sanity check for duplicates
    list(REMOVE_DUPLICATES LINKED_LIBS)

    # Create the testing executable target and link the libraries
    add_executable(${TESTNAME} ${SOURCE_FILES})
    target_link_libraries(${TESTNAME} ${LINKED_LIBS})

    # Perform the final GoogleTest boilerplate steps
    gtest_discover_tests(${TESTNAME}
            WORKING_DIRECTORY ${PROJECT_DIR}
            PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "${PROJECT_DIR}"
            )
    set_target_properties(${TESTNAME} PROPERTIES FOLDER test)
endfunction()

########################## Individual tests ##########################

########################## GeoJSON Unit Tests
add_test(test_geojson
        4
        geojson/JSONProperty_Test.cpp
        geojson/JSONGeometry_Test.cpp
        geojson/Feature_Test.cpp
        geojson/FeatureCollection_Test.cpp
        NGen::geojson
        )

########################## GeoPackage Unit Tests
if(NGEN_WITH_SQLITE3)
add_test(test_geopackage
        3
        geopackage/WKB_Test.cpp
        geopackage/SQLite_Test.cpp
        geopackage/GeoPackage_Test.cpp
        NGen::geopackage)
endif()

########################## Realization Config Unit Tests
add_test(test_realization_config
        1
        realizations/Formulation_Manager_Test.cpp
        NGen::core
        NGen::realizations_catchment
        NGen::core_mediator
        NGen::forcing
        libudunits2
        testbmicppmodel
        ${NETCDF_LIBRARIES}
)

########################## Nexus Tests
add_test(
        test_nexus
        1
        core/nexus/NexusTests.cpp
        NGen::core_nexus
)

########################## MPI Remote Nexus Tests
if(MPI_ACTIVE)
   add_test(
           test_remote_nexus
           1
           core/nexus/NexusRemoteTests.cpp
           NGen::core_nexus
   )
endif()

########################## Partitioning Tests
#if(MPI_ACTIVE)
   #TODO this test depends on a "reference hydrofabric" that is rather large
   #and the path to that reference is absolute in the test code.  Need to rethink this.
   add_test(
           test_partition
           1
           utils/Partition_Test.cpp
           NGen::core
           NGen::geojson
)
#endif()

########################## BMI C++ Tests
add_test(
        test_bmi_cpp
        2
        realizations/catchments/Bmi_Cpp_Adapter_Test.cpp
        realizations/catchments/Bmi_Cpp_Formulation_Test.cpp
        NGen::core
        NGen::realizations_catchment
        NGen::core_mediator
        NGen::forcing
        libudunits2
        ${NETCDF_LIBRARIES}
)

########################## BMI C Tests
if(BMI_C_LIB_ACTIVE)
    add_test(
            test_bmi_c
            2
            realizations/catchments/Bmi_C_Adapter_Test.cpp
            realizations/catchments/Bmi_C_Formulation_Test.cpp
            NGen::core
            NGen::realizations_catchment
            NGen::core_mediator
            NGen::forcing
            libudunits2
            ${NETCDF_LIBRARIES}
    )
endif()

########################## BMI Fortran Tests
if (${BMI_FORTRAN_ACTIVE})
    add_test(
            test_bmi_fortran
            2
            realizations/catchments/Bmi_Fortran_Adapter_Test.cpp
            realizations/catchments/Bmi_Fortran_Formulation_Test.cpp
            NGen::core
            NGen::realizations_catchment
            NGen::core_mediator
            NGen::forcing
            libudunits2
            ${NETCDF_LIBRARIES}
    )
endif()

########################## BMI Python Tests
if (${NGEN_ACTIVATE_PYTHON})
    add_test(
            test_bmi_python
            2
            realizations/catchments/Bmi_Py_Adapter_Test.cpp
            realizations/catchments/Bmi_Py_Formulation_Test.cpp
            NGen::core
            NGen::realizations_catchment
            NGen::core_mediator
            NGen::forcing
            libudunits2
            ${NETCDF_LIBRARIES}
    )
endif()

########################## BMI Multi Tests
add_test(
        test_bmi_multi
        2
        realizations/catchments/Bmi_Multi_Formulation_Test.cpp
        realizations/catchments/Bmi_Cpp_Multi_Array_Test.cpp
        NGen::core
        NGen::realizations_catchment
        NGen::core_mediator
        NGen::forcing
        libudunits2
        ${NETCDF_LIBRARIES}
)

########################## All BMI Unit Tests
if(BMI_C_LIB_ACTIVE)
    if (${BMI_FORTRAN_ACTIVE})
        if (${NGEN_ACTIVATE_PYTHON})
            add_test(
                    test_bmi_unit_all
                    7
                    realizations/catchments/Bmi_C_Adapter_Test.cpp
                    realizations/catchments/Bmi_C_Formulation_Test.cpp
                    realizations/catchments/Bmi_Fortran_Adapter_Test.cpp
                    realizations/catchments/Bmi_Fortran_Formulation_Test.cpp
                    realizations/catchments/Bmi_Py_Adapter_Test.cpp
                    realizations/catchments/Bmi_Py_Formulation_Test.cpp
                    realizations/catchments/Bmi_Multi_Formulation_Test.cpp
                    NGen::core
                    NGen::realizations_catchment
                    NGen::core_mediator
                    NGen::forcing
                    libudunits2
                    ${NETCDF_LIBRARIES}
            )
        endif()
    endif()
endif()

########################## Comparison tests for the BMI PET implementation
# TODO: this probably needs to be added to integration testing also
add_test(
        compare_pet
        1
        realizations/catchments/Bmi_C_Pet_IT.cpp
        NGen::core
        NGen::realizations_catchment
        NGen::core_mediator
        NGen::forcing
        libudunits2
        ${NETCDF_LIBRARIES}
)

########################## Internal Torchlib LSTM Tests
if(LSTM_TORCH_LIB_ACTIVE)
    add_test(
            test_lstm
            2
            models/lstm/include/LSTM_Test.cpp
            realizations/catchments/LSTM_Realization_Test.cpp
            NGen::core
            NGen::realizations_catchment
            NGen::core_mediator
            libudunits2
            ${NETCDF_LIBRARIES}
    )
endif()

########################## Routing Tests
if(NGEN_ACTIVATE_ROUTING)
    add_test(
            test_routing_pybind
            1
            routing/Routing_Py_Bind_Test.cpp
            NGen::core # for filechecker utility
            NGen::routing
            pybind11::embed
    )
endif()

########################## Network Class Tests
add_test(
        test_network
        1
        core/NetworkTests.cpp
        NGen::core
        NGen::geojson
)

########################### Netcdf Forcing Tests
#if(NETCDF_ACTIVE)
add_test(
        test_netcdf_forcing
        1
        forcing/NetCDFPerFeatureDataProvider_Test.cpp
        NGen::core
        NGen::core_nexus
        NGen::core_mediator
        NGen::forcing
        NGen::geojson
        NGen::realizations_catchment
        libudunits2
        ${NETCDF_LIBRARIES}
)
#endif()

########################## Primary Combined Unit Test Target
add_test(
        test_unit
        13
        geojson/JSONProperty_Test.cpp
        geojson/JSONGeometry_Test.cpp
        geojson/Feature_Test.cpp
        geojson/FeatureCollection_Test.cpp
        forcing/CsvPerFeatureForcingProvider_Test.cpp
        forcing/OptionalWrappedDataProvider_Test.cpp
        forcing/NetCDFPerFeatureDataProvider_Test.cpp
        core/mediator/UnitsHelper_Tests.cpp
        simulation_time/Simulation_Time_Test.cpp
        core/NetworkTests.cpp
        utils/include/StreamOutputTest.cpp
        realizations/Formulation_Manager_Test.cpp
        utils/Partition_Test.cpp
        NGen::core
        NGen::core_nexus
        NGen::core_mediator
        NGen::forcing
        NGen::geojson
        NGen::realizations_catchment
        libudunits2
        testbmicppmodel
        ${NETCDF_LIBRARIES}
)

# All automated tests
add_test(
        test_all
        12
        geojson/JSONProperty_Test.cpp
        geojson/JSONGeometry_Test.cpp
        geojson/Feature_Test.cpp
        geojson/FeatureCollection_Test.cpp
        forcing/CsvPerFeatureForcingProvider_Test.cpp
        forcing/OptionalWrappedDataProvider_Test.cpp
        forcing/NetCDFPerFeatureDataProvider_Test.cpp
        simulation_time/Simulation_Time_Test.cpp
        core/mediator/UnitsHelper_Tests.cpp
        realizations/Formulation_Manager_Test.cpp
        core/nexus/NexusTests.cpp
        utils/Partition_Test.cpp
        NGen::core
        gmock
        NGen::core_nexus
        NGen::core_mediator
        NGen::forcing
        NGen::geojson
        NGen::realizations_catchment
        libudunits2
        testbmicppmodel
        ${NETCDF_LIBRARIES}
)
