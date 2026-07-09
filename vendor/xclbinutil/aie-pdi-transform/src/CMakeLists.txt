# CMakeLists.txt in src/

# -fvisibility is a GCC/Clang-driver flag with no MSVC-driver equivalent (symbol
# visibility is not a concept for this static lib on Windows). CMake sets MSVC=1 for
# both cl and clang-cl, so this correctly skips it for either Windows compiler; without
# the guard cl warns "D9002: ignoring unknown option '-fvisibility=default'".
if(NOT MSVC)
  add_compile_options(-fvisibility=default)
endif()

# This builds a static library libtransformcdo, which 
# includes all the *.c files in lib/
add_library( transformcdo STATIC
    pdi-parsing.c
    cdo-parsing.c
    pdi-transform.c
    ../lib/cdo_cmd.c
    ../lib/load_pdi.c
    ../lib/pdi_transform.c
)

# Set C flags
# set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wextra -fvisibility=default")

target_include_directories(transformcdo PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/../libinclude
    ${CMAKE_CURRENT_SOURCE_DIR}
)

