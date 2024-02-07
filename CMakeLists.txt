cmake_minimum_required(VERSION 3.10)
project(ConjugateGradients VERSION 1.0 DESCRIPTION "Conjugate Gradients Solver for HPC" LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Compiler options for better code quality
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    add_compile_options(-Wall -Wextra -pedantic -O2)
endif()

# Add executables
add_executable(conjugate_gradients conjugate_gradients-main/src/conjugate_gradients.cpp)
target_include_directories(conjugate_gradients PRIVATE ${PROJECT_SOURCE_DIR}/conjugate_gradients-main/src)

add_executable(random_spd_system conjugate_gradients-main/src/random_spd_system.cpp)
target_include_directories(random_spd_system PRIVATE ${PROJECT_SOURCE_DIR}/conjugate_gradients-main/src)

# Attempt to find MKL
find_package(MKL REQUIRED)
if(MKL_FOUND)
    target_include_directories(random_spd_system PRIVATE ${MKL_INCLUDE_DIRS})
    target_link_libraries(random_spd_system PRIVATE ${MKL_LIBRARIES})
endif()

# Options for enabling/disabling parallelization libraries
option(USE_OPENMP "Enable OpenMP support" OFF)
option(USE_MPI "Enable MPI support" OFF)
option(USE_CUDA "Enable CUDA support" OFF)

# Find and link parallelization libraries if enabled and found
if(USE_OPENMP)
    find_package(OpenMP)
    if(OpenMP_CXX_FOUND)
        target_link_libraries(conjugate_gradients PUBLIC OpenMP::OpenMP_CXX)
    endif()
endif()

if(USE_MPI)
    find_package(MPI)
    if(MPI_CXX_FOUND)
        target_link_libraries(conjugate_gradients PUBLIC MPI::MPI_CXX)
    endif()
endif()

if(USE_CUDA)
    find_package(CUDA)
    if(CUDA_FOUND)
        target_link_libraries(conjugate_gradients PUBLIC CUDA::cuda_driver)
    endif()
endif()