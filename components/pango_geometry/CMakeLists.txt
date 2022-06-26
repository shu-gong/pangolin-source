get_filename_component(COMPONENT ${CMAKE_CURRENT_LIST_DIR} NAME)

find_package(Eigen3 REQUIRED NO_MODULE)
if (Eigen3_FOUND)
    message("-- Found Eigen version ${Eigen3_VERSION}: ${Eigen3_DIR}")
    if (CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64.*|AARCH64.*)" AND
            Eigen3_VERSION VERSION_LESS 3.3.4)
        # As per issue #289: https://github.com/ceres-solver/ceres-solver/issues/289
        # the bundle_adjustment_test will fail for Eigen < 3.3.4 on aarch64.
        message(FATAL_ERROR "-- Ceres requires Eigen version >= 3.3.4 on aarch64. "
                "Detected version of Eigen is: ${Eigen3_VERSION}.")
    endif()

    if (EIGENSPARSE)
        message("-- Enabling use of Eigen as a sparse linear algebra library.")
        list(APPEND CERES_COMPILE_OPTIONS CERES_USE_EIGEN_SPARSE)
    else (EIGENSPARSE)
        message("-- Disabling use of Eigen as a sparse linear algebra library.")
        message("   This does not affect the covariance estimation algorithm ")
        message("   which can still use the EIGEN_SPARSE_QR algorithm.")
        add_definitions(-DEIGEN_MPL2_ONLY)
    endif (EIGENSPARSE)
endif (Eigen3_FOUND)
#message(STATUS "Found Eigen: '${EIGEN3_INCLUDE_DIRS}'")
target_compile_definitions(${COMPONENT} PUBLIC HAVE_EIGEN)

target_sources( ${COMPONENT}
PRIVATE
    ${CMAKE_CURRENT_LIST_DIR}/src/geometry.cpp
    ${CMAKE_CURRENT_LIST_DIR}/src/geometry_obj.cpp
    ${CMAKE_CURRENT_LIST_DIR}/src/geometry_ply.cpp
)

target_link_libraries(${COMPONENT} pango_core pango_image tinyobj Eigen3::Eigen)
target_include_directories(${COMPONENT} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
install(DIRECTORY "${CMAKE_CURRENT_LIST_DIR}/include"
  DESTINATION ${CMAKE_INSTALL_PREFIX}
)
