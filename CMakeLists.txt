cmake_minimum_required(VERSION 2.8)
project (libpng)

if (DEFINED CMAKE_TOOLCHAIN_FILE)
	message("Include Cross Platform cmake: " ${CROSS_PLATFORM_CMAKE})
	get_filename_component(COMPILER_NAME ${CMAKE_C_COMPILER} NAME)
	message( "CMAKE_C_COMPILER: ${COMPILER_NAME}")
	string(REGEX REPLACE "(.*)-gcc" "\\1" target_platform "${COMPILER_NAME}")
	message( "target_platform: ${target_platform}")
	set(target_platform_preconfigure_options "CC=${target_platform}-gcc;AR=${target_platform}-ar;LD=${target_platform}-ld;RANLIB=${target_platform}-ranlib;NM=${target_platform}-nm;AS=${target_platform}-as;OBJDUMP=${target_platform}-objdump;OBJCOPY=${target_platform}-objcopy;STRIP=${target_platform}-strip;STRINGS=${target_platform}-strings;SIZE=${target_platform}-size")
endif(DEFINED CMAKE_TOOLCHAIN_FILE)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

set(SELF_CPPFLAGS "")
set(SELF_LDFLAGS "")
set(SELF_DEPENDS "")

add_subdirectory(libz-cmake)
set(SELF_CPPFLAGS "${SELF_CPPFLAGS};-I${CMAKE_BINARY_DIR}/zlib-1.2.7")
set(SELF_LDFLAGS "${SELF_LDFLAGS};-L${CMAKE_BINARY_DIR}/lib" )
set(SELF_DEPENDS "${SELF_DEPENDS};build_libz")

set(TOOL_LIBPNG_SOURCE libpng-1.6.1.tar.xz)
set(TOOL_LIBPNG_DIR libpng-1.6.1)
set(CONFIG_ADD_OPTIONS "--enable-static;--enable-shared")
	if(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT BUILD_LIBPNG
			COMMAND wget -qO- http://downloads.sourceforge.net/project/libpng/libpng16/1.6.1/${TOOL_LIBPNG_SOURCE} | tar -xJvf - && cd ${CMAKE_BINARY_DIR}/${TOOL_LIBPNG_DIR} && ${target_platform_preconfigure_options} ./configure --host="${target_platform}" LDFLAGS="${SELF_LDFLAGS}" CPPFLAGS="${SELF_CPPFLAGS}" ${CONFIG_ADD_OPTIONS} && make -j2 && mkdir -p ${CMAKE_LIBRARY_OUTPUT_DIRECTORY} && cp -l ${CMAKE_BINARY_DIR}/${TOOL_LIBPNG_DIR}/.libs/lib* ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}
			WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
			COMMENT "Build libpng"
		)
	else(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT BUILD_LIBPNG
			COMMAND wget -qO- http://downloads.sourceforge.net/project/libpng/libpng16/1.6.1/${TOOL_LIBPNG_SOURCE} | tar -xJvf - && cd ${CMAKE_BINARY_DIR}/${TOOL_LIBPNG_DIR} && ./configure LDFLAGS="${SELF_LDFLAGS}" CPPFLAGS="${SELF_CPPFLAGS}" ${CONFIG_ADD_OPTIONS} && make -j2 && mkdir -p ${CMAKE_LIBRARY_OUTPUT_DIRECTORY} && cp -l ${CMAKE_BINARY_DIR}/${TOOL_LIBPNG_DIR}/.libs/lib* ${CMAKE_LIBRARY_OUTPUT_DIRECTORY} 
			WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
			COMMENT "Build libpng" 
		)
	endif(DEFINED CMAKE_TOOLCHAIN_FILE)
	add_custom_target(build_libpng ALL DEPENDS ${SELF_DEPENDS} BUILD_LIBPNG )
