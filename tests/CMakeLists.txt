include(CheckCSourceCompiles)

set(find_standard_math_library_test_program
"#include <math.h>
int main() { sin(0.0); log(0.0f); }")

# first try compiling/linking the test program without any linker flags

set(CMAKE_REQUIRED_FLAGS "")
set(CMAKE_REQUIRED_LIBRARIES "")
CHECK_C_SOURCE_COMPILES(
  "${find_standard_math_library_test_program}"
  standard_math_library_linked_to_automatically
)

if(standard_math_library_linked_to_automatically)

  # the test program linked successfully without any linker flag.
  set(STANDARD_MATH_LIBRARY "")
  set(STANDARD_MATH_LIBRARY_FOUND TRUE)

else()

  # the test program did not link successfully without any linker flag.
  # This is a very uncommon case that so far we only saw on QNX. The next try is the
  # standard name 'm' for the standard math library.

  set(CMAKE_REQUIRED_LIBRARIES "m")
  CHECK_C_SOURCE_COMPILES(
    "${find_standard_math_library_test_program}"
    standard_math_library_linked_to_as_m)

  if(standard_math_library_linked_to_as_m)

    # the test program linked successfully when linking to the 'm' library
    set(STANDARD_MATH_LIBRARY "m")
    set(STANDARD_MATH_LIBRARY_FOUND TRUE)

  else()

    # the test program still doesn't link successfully
    set(STANDARD_MATH_LIBRARY_FOUND FALSE)

  endif()

endif()



if (BUILD_TESTING)

enable_testing()
include(Dart)

add_executable(opusapi test_opus_api.c)
add_executable(opusenc test_opus_encode.c opus_encode_regressions.c)
add_executable(opusdec test_opus_decode.c)
add_executable(opus_padding test_opus_padding.c)
add_executable(opus_demo ../src/opus_demo.c)

target_link_libraries(opusapi opus ${STANDARD_MATH_LIBRARY})
target_link_libraries(opusenc opus ${STANDARD_MATH_LIBRARY})
target_link_libraries(opusdec opus ${STANDARD_MATH_LIBRARY})
target_link_libraries(opus_padding opus ${STANDARD_MATH_LIBRARY})
target_link_libraries(opus_demo opus ${STANDARD_MATH_LIBRARY})

add_test(api opusapi)
add_test(enc opusenc)
add_test(dec opusdec)
add_test(pad opus_padding)

endif (BUILD_TESTING)

