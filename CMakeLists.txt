cmake_minimum_required(VERSION 3.10)

set (CMAKE_CONFIGURATION_TYPES "Debug;Release")

project(lwIP)

set(LWIP_TIMEVAL_PRIVATE 0 CACHE STRING "set this to 0 and include")
set(LWIP_BUILD_SAMPLE OFF CACHE BOOL "Enable platform example")
set(CMAKE_INSTALL_PREFIX "/" CACHE STRING "Install Prefix")


# Example lwIP application
set(LWIP_DIR ${CMAKE_CURRENT_SOURCE_DIR})

set(LWIP_DEFINITIONS
	LWIP_DEBUG=1
	LWIP_TIMEVAL_PRIVATE=${LWIP_TIMEVAL_PRIVATE}
)

if ( LWIP_BUILD_SAMPLE )
	if (${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
	  add_subdirectory(${LWIP_DIR}/contrib/ports/win32/example_app)
	elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux" OR ${CMAKE_SYSTEM_NAME} STREQUAL "Darwin")
	  add_subdirectory(${LWIP_DIR}/contrib/ports/unix/example_app)
	else()
	  message(WARNING "Host ${CMAKE_SYSTEM_NAME} is not supported to build example_app")
	endif()
endif()

# Source package generation
set(CPACK_SOURCE_GENERATOR "ZIP")
set(CPACK_SOURCE_PACKAGE_DESCRIPTION_SUMMARY "lwIP lightweight IP stack")
set(CPACK_PACKAGE_VERSION_MAJOR "${LWIP_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${LWIP_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${LWIP_VERSION_REVISION}")
set(CPACK_SOURCE_IGNORE_FILES "/build/;${CPACK_SOURCE_IGNORE_FILES};.git")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "lwip-${LWIP_VERSION_MAJOR}.${LWIP_VERSION_MINOR}.${LWIP_VERSION_REVISION}")
include(CPack)

# Generate docs before creating source package
include(src/Filelists.cmake)
add_custom_target(dist COMMAND ${CMAKE_MAKE_PROGRAM} package_source)
if (TARGET lwipdocs)
  add_dependencies(dist lwipdocs)
endif()


add_subdirectory(src/core build-lwipcore)

# install cmake package

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
        "${CMAKE_CURRENT_BINARY_DIR}/lwip/lwipConfigVersion.cmake"
        VERSION "${LWIP_VERSION_MAJOR}.${LWIP_VERSION_MINOR}.${LWIP_VERSION_REVISION}"
        COMPATIBILITY AnyNewerVersion
)

export(EXPORT lwipTargets
        FILE "${CMAKE_CURRENT_BINARY_DIR}/lwip/lwipTargets.cmake"
        NAMESPACE lwip::
)

set(ConfigPackageLocation lib/cmake/lwip)

install(EXPORT lwipTargets
	FILE
		lwipTargets.cmake
	NAMESPACE
		lwip::
	DESTINATION
		${ConfigPackageLocation}
)

install(
	FILES
		lwipConfig.cmake
		"${CMAKE_CURRENT_BINARY_DIR}/lwip/lwipConfigVersion.cmake"
	DESTINATION
		${ConfigPackageLocation}
	COMPONENT
		Devel
)


