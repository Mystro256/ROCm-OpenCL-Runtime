cmake_minimum_required(VERSION 3.5.1)
project(rocm-opencl)

set(CPACK_COMPONENTS_ALL binary dev icd)
set(CPACK_DEB_COMPONENT_INSTALL ON)
set(CPACK_RPM_COMPONENT_INSTALL ON)

install(TARGETS clinfo DESTINATION bin COMPONENT binary)
install(TARGETS amdocl DESTINATION lib COMPONENT binary)
install(FILES ${CMAKE_SOURCE_DIR}/LICENSE.txt DESTINATION share/doc/${CMAKE_PROJECT_NAME} COMPONENT binary)

install(DIRECTORY ${CMAKE_SOURCE_DIR}/khronos/headers/opencl2.2/CL
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR} COMPONENT dev
        USE_SOURCE_PERMISSIONS
        PATTERN cl_d3d10.h EXCLUDE
        PATTERN cl_d3d11.h EXCLUDE
        PATTERN cl_dx9_media_sharing.h EXCLUDE
        PATTERN cl_egl.h EXCLUDE )

install(TARGETS OpenCL DESTINATION lib COMPONENT icd )
install(FILES ${CMAKE_SOURCE_DIR}/khronos/icd/LICENSE DESTINATION share/doc/rocm-ocl-icd COMPONENT icd)


# Generic CPACK variables
set(CPACK_GENERATOR "DEB;RPM" CACHE STRING "Package types to build")
set(CPACK_PACKAGE_CONTACT "ROCm OpenCL Support <rocm-opencl.support@amd.com>")
set(CPACK_PACKAGE_VENDOR "Advanced Micro Devices, Inc.")
set(CPACK_PACKAGE_DESCRIPTION "OpenCL: Open Computing Language on ROCclr")




#########Binary-Package###############
set(CPACK_DEBIAN_BINARY_PACKAGE_NAME "rocm-opencl")
set(CPACK_RPM_BINARY_PACKAGE_NAME "rocm-opencl")

# Debian CPACK variables
set(CPACK_BINARY_DEB "ON")
if(DEFINED ENV{CPACK_DEBIAN_PACKAGE_RELEASE})
  set(CPACK_DEBIAN_PACKAGE_RELEASE $ENV{CPACK_DEBIAN_PACKAGE_RELEASE})
else()
  set(CPACK_DEBIAN_PACKAGE_RELEASE "local")
endif()
message("Using CPACK_DEBIAN_PACKAGE_RELEASE ${CPACK_DEBIAN_PACKAGE_RELEASE}")

set(CPACK_DEBIAN_FILE_NAME "DEB-DEFAULT")
set(CPACK_DEBIAN_BINARY_PACKAGE_DEPENDS "rocm-ocl-icd, libelf-dev, comgr, hsa-rocr, rocm-core")

# RPM CPACK variables
set(CPACK_BINARY_RPM "ON")
set(CPACK_RPM_BINARY_PACKAGE_REQUIRES "rocm-ocl-icd, comgr, hsa-rocr, rocm-core")

#Unable to set CPACK_RPM_BIANRY_PACKAGE_LICENSE to control individual pacakge license
#Hence combining the license for BINARY,DEV,ICD to one
set(CPACK_RPM_PACKAGE_LICENSE "MIT and ASL 2.0")

#########Dev-Package###############

# DEBIAN CPACK variables
set(CPACK_DEBIAN_DEV_PACKAGE_NAME "rocm-opencl-dev")
set(CPACK_DEBIAN_DEV_PACKAGE_DEPENDS "mesa-common-dev, rocm-opencl, hsa-rocr-dev, rocm-core")
# RPM CPACK variables
set(CPACK_RPM_DEV_PACKAGE_NAME "rocm-opencl-devel")
set(CPACK_RPM_DEV_PACKAGE_REQUIRES "rocm-opencl, hsa-rocr-devel, rocm-core")


#############################
# ICD Loader
#############################

# Debian CPACK variables
set(CPACK_ICD_DEB "ON")
set(CPACK_DEBIAN_ICD_PACKAGE_NAME "rocm-ocl-icd")

set(CPACK_DEBIAN_ICD_PACKAGE_CONTROL_EXTRA "${CMAKE_BINARY_DIR}/packages/rocm-ocl-icd/postinst;${CMAKE_BINARY_DIR}/packages/rocm-ocl-icd/prerm")
set(CPACK_DEBIAN_ICD_PACKAGE_DEPENDS "rocm-core")

# RPM CPACK variables
set(CPACK_ICD_RPM "ON")
set(CPACK_RPM_ICD_PACKAGE_NAME "rocm-ocl-icd")



set(CPACK_RPM_ICD_POST_INSTALL_SCRIPT_FILE "${CMAKE_BINARY_DIR}/packages/rocm-ocl-icd/rpm_post")
set(CPACK_RPM_ICD_POST_UNINSTALL_SCRIPT_FILE "${CMAKE_BINARY_DIR}/packages/rocm-ocl-icd/rpm_postun")
set(CPACK_RPM_ICD_PACKAGE_REQUIRES "rocm-core")

if(DEFINED ENV{CPACK_RPM_PACKAGE_RELEASE})
  set(CPACK_RPM_PACKAGE_RELEASE $ENV{CPACK_RPM_PACKAGE_RELEASE})
else()
  set(CPACK_RPM_PACKAGE_RELEASE "local")
endif()
message("Using CPACK_RPM_PACKAGE_RELEASE ${CPACK_RPM_PACKAGE_RELEASE}")


set(CPACK_RPM_FILE_NAME "RPM-DEFAULT")
set(CPACK_RPM_PACKAGE_AUTOREQPROV " no")

## 'dist' breaks manual builds on debian systems due to empty Provides
execute_process(COMMAND rpm --eval %{?dist}
                 RESULT_VARIABLE PROC_RESULT
                 OUTPUT_VARIABLE EVAL_RESULT
                 OUTPUT_STRIP_TRAILING_WHITESPACE)
message("RESULT_VARIABLE ${PROC_RESULT} OUTPUT_VARIABLE: ${EVAL_RESULT}")

if(PROC_RESULT EQUAL "0" AND NOT EVAL_RESULT STREQUAL "")
  string(APPEND CPACK_RPM_PACKAGE_RELEASE "%{?dist}")
endif()
# Remove dependency on rocm-core if -DROCM_DEP_ROCMCORE=ON not given to cmake
if(NOT ROCM_DEP_ROCMCORE)
    string(REGEX REPLACE ",? ?rocm-core" "" CPACK_RPM_BINARY_PACKAGE_REQUIRES ${CPACK_RPM_BINARY_PACKAGE_REQUIRES})
    string(REGEX REPLACE ",? ?rocm-core" "" CPACK_DEBIAN_BINARY_PACKAGE_DEPENDS ${CPACK_DEBIAN_BINARY_PACKAGE_DEPENDS})
    string(REGEX REPLACE ",? ?rocm-core" "" CPACK_RPM_DEV_PACKAGE_REQUIRES ${CPACK_RPM_DEV_PACKAGE_REQUIRES})
    string(REGEX REPLACE ",? ?rocm-core" "" CPACK_DEBIAN_DEV_PACKAGE_DEPENDS ${CPACK_DEBIAN_DEV_PACKAGE_DEPENDS})
    string(REGEX REPLACE ",? ?rocm-core" "" CPACK_RPM_ICD_PACKAGE_REQUIRES ${CPACK_RPM_ICD_PACKAGE_REQUIRES})
    string(REGEX REPLACE ",? ?rocm-core" "" CPACK_DEBIAN_ICD_PACKAGE_DEPENDS ${CPACK_DEBIAN_ICD_PACKAGE_DEPENDS})
endif()

include(CPack)
