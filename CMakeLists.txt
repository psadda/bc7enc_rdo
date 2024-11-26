project(bc7enc)

cmake_minimum_required(VERSION 3.19)

add_executable(bc7enc)

target_sources(bc7enc PRIVATE
	bc7enc.cpp
	bc7enc.h
	bc7decomp.cpp
	bc7decomp_ref.cpp
	bc7decomp.h
	lodepng.cpp
	lodepng.h
	test.cpp
	rgbcx.cpp
	rgbcx.h
	rgbcx_table4.h
	rgbcx_table4_small.h
	utils.cpp
	utils.h
	miniz.h
	dds_defs.h
	ert.cpp
	ert.h
	rdo_bc_encoder.cpp
	rdo_bc_encoder.h
)

find_program(ISPC_COMPILER ispc)

if(MSVC AND NOT CMAKE_CXX_COMPILER_ID STREQUAL Clang)
	set(OBJECT_EXT obj)
else()
	set(OBJECT_EXT o)
endif()

if(ISPC_COMPILER)
	message(STATUS "BC7ENC: Found Intel ISPC. BC7 support is enabled.")
	add_custom_command(
		OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/bc7e.${OBJECT_EXT}
		       ${CMAKE_CURRENT_BINARY_DIR}/bc7e_avx.${OBJECT_EXT}
		       ${CMAKE_CURRENT_BINARY_DIR}/bc7e_sse2.${OBJECT_EXT}
		       ${CMAKE_CURRENT_BINARY_DIR}/bc7e_sse4.${OBJECT_EXT}
		       ${CMAKE_CURRENT_BINARY_DIR}/bc7e_avx2.${OBJECT_EXT}
		       ${CMAKE_CURRENT_BINARY_DIR}/bc7e_ispc.h
		COMMAND ${ISPC_COMPILER} -O3
		       ${CMAKE_CURRENT_SOURCE_DIR}/bc7e.ispc
		       -o ${CMAKE_CURRENT_BINARY_DIR}/bc7e.${OBJECT_EXT}
		       -h ${CMAKE_CURRENT_BINARY_DIR}/bc7e_ispc.h
		       --target=sse2,sse4,avx,avx2 --opt=fast-math --opt=disable-assertions
		DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/bc7e.ispc
	)
	target_sources(bc7enc PRIVATE
		${CMAKE_CURRENT_BINARY_DIR}/bc7e.${OBJECT_EXT}
		${CMAKE_CURRENT_BINARY_DIR}/bc7e_avx.${OBJECT_EXT}
		${CMAKE_CURRENT_BINARY_DIR}/bc7e_avx2.${OBJECT_EXT}
		${CMAKE_CURRENT_BINARY_DIR}/bc7e_sse2.${OBJECT_EXT}
		${CMAKE_CURRENT_BINARY_DIR}/bc7e_sse4.${OBJECT_EXT}
	)
	target_compile_definitions(bc7enc PRIVATE SUPPORT_BC7E)
	target_include_directories(bc7enc PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
else()
	message("BC7ENC: Could not find Intel ISPC. BC7 support is NOT enabled. (Try manually setting ISPC_COMPILER.)")
endif()

if(MSVC)
	target_compile_options(bc7enc PRIVATE /openmp /W4)
else()
	target_compile_options(bc7enc PRIVATE -fopenmp -fno-strict-aliasing -Wall -Wextra)
	target_link_libraries(bc7enc m)
endif()
