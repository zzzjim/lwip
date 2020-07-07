
# lwIP libraries

configure_file("../../lwipopts.h.in" "${CMAKE_CURRENT_BINARY_DIR}/lwipopts.h")

if ( UNIX )
	set(LWIP_PLATFORM_INCLUDES "../../contrib/ports/unix/port/include/")
	set(LWIP_EXTRA_SRS "../../contrib/ports/unix/port/sys_arch.c")
else()
	set(LWIP_PLATFORM_INCLUDES "../../contrib/ports/win32/include/")
	set(LWIP_EXTRA_SRS "../../contrib/ports/win32/sys_arch.c")
endif()

add_library(lwipcore STATIC
	${lwipcore_SRCS}
	${lwipcore4_SRCS}
	${lwipcore6_SRCS}
	${lwipapi_SRCS}
	${lwipnetif_SRCS}
	${LWIP_EXTRA_SRS}
)
target_compile_options(lwipcore PRIVATE ${LWIP_COMPILER_FLAGS})
target_compile_definitions(lwipcore PRIVATE ${LWIP_DEFINITIONS} ${LWIP_MBEDTLS_DEFINITIONS})
target_include_directories(lwipcore
	PRIVATE
		../include/
		${CMAKE_CURRENT_BINARY_DIR}
		${LWIP_PLATFORM_INCLUDES}
		${LWIP_MBEDTLS_INCLUDE_DIRS}
)

file(GLOB LWIP_PUBLIC_HEADERS "../include/lwip/*.h")

set(LWIP_PUBLIC_HEADERS
	${LWIP_PUBLIC_HEADERS}
	"${CMAKE_CURRENT_BINARY_DIR}/lwipopts.h"
)

set_target_properties(lwipcore PROPERTIES PUBLIC_HEADER "${LWIP_PUBLIC_HEADERS}")

install(
        TARGETS lwipcore
        EXPORT lwipTargets
        LIBRARY DESTINATION "lib"
        PUBLIC_HEADER DESTINATION "include/lwip"
)

install(
	DIRECTORY ${LWIP_PLATFORM_INCLUDES}/arch
	DESTINATION "include/lwip"
)

install(
	DIRECTORY "../include/lwip/priv"
	DESTINATION "include/lwip"
)
