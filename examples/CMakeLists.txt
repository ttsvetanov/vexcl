function(add_vexcl_example example)
    add_executable(${example} ${example}.cpp)
    target_link_libraries(${example} ${BACKEND_LIBS} ${Boost_LIBRARIES})

    list(APPEND binaries "${CMAKE_CURRENT_BINARY_DIR}/${example}")
    list(APPEND sources  "${CMAKE_CURRENT_SOURCE_DIR}/${example}.cpp")
endfunction()

#----------------------------------------------------------------------------
# General examples
#----------------------------------------------------------------------------
add_vexcl_example(devlist)
add_vexcl_example(benchmark)
if ("${VEXCL_BACKEND}" STREQUAL "CUDA")
    target_link_libraries(benchmark ${CUDA_cusparse_LIBRARY})
endif()

if ("${VEXCL_BACKEND}" STREQUAL "OpenCL")
    add_vexcl_example(exclusive)
endif()

if (CLOGS)
    target_link_libraries(benchmark ${CLOGS_LIB})
endif()

find_path(MBA_INCLUDE mba/mba.hpp)
if (MBA_INCLUDE)
    include_directories(${MBA_INCLUDE})
    add_vexcl_example(mba_benchmark)
endif ()

#----------------------------------------------------------------------------
# Symbolic example uses Boost.odeint available since Boost v1.53
#----------------------------------------------------------------------------
if (NOT ${Boost_VERSION} STRLESS "105300")
    add_vexcl_example(symbolic)
endif ()

#----------------------------------------------------------------------------
# FFT benchmark
#----------------------------------------------------------------------------
add_vexcl_example(fft_profile)
add_vexcl_example(fft_benchmark)
find_package(FFTW QUIET)
if (FFTW_FOUND)
    include_directories(${FFTW_INCLUDES})
    add_definitions(-DHAVE_FFTW)
    set(FFT_BENCHMARK_LIBS ${FFT_BENCHMARK_LIBS} ${FFTW_LIBRARIES})
endif ()

find_package(CUDA QUIET)
if(CUDA_FOUND)
    include_directories(${CUDA_INCLUDE_DIRS})
    add_definitions(-DHAVE_CUDA)
    set(FFT_BENCHMARK_LIBS ${FFT_BENCHMARK_LIBS} ${CUDA_LIBRARIES} ${CUDA_cufft_LIBRARY})
endif()

target_link_libraries(fft_benchmark ${FFT_BENCHMARK_LIBS})

#----------------------------------------------------------------------------
# Thrust interoperation examples
#----------------------------------------------------------------------------
if ("${VEXCL_BACKEND}" STREQUAL "CUDA")
    set(CUDA_OPTIONS "-arch=sm_${CUDA_COMPUTE_CAPABILITY}")
    if (NOT WIN32)
	set(CUDA_PROPAGATE_HOST_FLAGS OFF)
        set(CUDA_OPTIONS "${CUDA_OPTIONS} -Xcompiler -std=c++03")
    endif()
    cuda_add_executable(thrust-sort thrust-sort.cpp thrust-sort.cu OPTIONS ${CUDA_OPTIONS})
    target_link_libraries(thrust-sort ${Boost_LIBRARIES} ${BACKEND_LIBS})
endif()

#----------------------------------------------------------------------------
# Install compiled examples and sources
#----------------------------------------------------------------------------
install(PROGRAMS ${binaries} DESTINATION share/vexcl/examples)
install(FILES    ${sources}  DESTINATION share/vexcl/examples/src)

#----------------------------------------------------------------------------
add_subdirectory(viennacl)

