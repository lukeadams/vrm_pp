cmake_minimum_required(VERSION 3.0)
project(vrm_pp)
#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME vrm_pp)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
	set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
	set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
	include(CPM)
else()
	set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
	find_package(Git)
	if(NOT GIT_FOUND)
		message(FATAL_ERROR "CPM requires Git.")
	endif()
	if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
		message(STATUS "Cloning repo (https://github.com/iauns/cpm)")
		execute_process(
			COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
			RESULT_VARIABLE error_code
			OUTPUT_QUIET ERROR_QUIET)
		if(error_code)
			message(FATAL_ERROR "CPM failed to get the hash for HEAD")
		endif()
	endif()
	include(${CPM_DIR}/CPM.cmake)
endif()

#-----------------------------------------------------------------------
# CPM deps
#-----------------------------------------------------------------------

# Include SSVCmake.
SET(SSVDepsRoot "https://github.com/lukeadams/")
CPM_AddModule("SSVCMake"
	GIT_REPOSITORY "${SSVDepsRoot}SSVCMake"
	GIT_TAG "cpm")
SET(CMAKE_MODULE_PATH ${CPM_KV_SOURCEDIR_MAP_SSVCMake}/cmake/modules ${CMAKE_MODULE_PATH})
include(SSVCMake)
CPM_InitModule(${CPM_MODULE_NAME})

#-----------------------------------------------------------------------
# Rest of file
#-----------------------------------------------------------------------

# Basic project setup.
vrm_cmake_init_project(vrm_pp)

# Alias the include directory.
set(VRM_PP_INC_DIR "${VRM_PP_SOURCE_DIR}/include/")

# Other compiler flags.
vrm_cmake_add_common_compiler_flags()

# Setup user options.
vrm_cmake_add_option_memcheck()
vrm_cmake_add_option_no_exceptions()
vrm_cmake_add_option_werror()

# The `check` target runs all tests and examples.
vrm_check_target()

# Setup subdirectories.
add_subdirectory(generator)
add_subdirectory(test)

# Create header-only install target (automatically glob)
vrm_cmake_header_only_install_glob("${VRM_PP_INC_DIR}" "include")
