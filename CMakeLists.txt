cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(vigenere-provider LANGUAGES C)
set(CMAKE_C_STANDARD 90)

include(CheckLibraryExists)
include(CheckFunctionExists)

enable_testing()

find_package(OpenSSL 3.0 REQUIRED)
include_directories(${OPENSSL_INCLUDE_DIR})
if (DEFINED MSVC_VERSION)
  # FindOpenSSL.cmake assumes http://www.slproweb.com/products/Win32OpenSSL.html
  # and gets it quote wrong when an install from OpenSSL source is present
  set(OPENSSL_CRYPTO_LIBRARY ${OPENSSL_ROOT_DIR}/lib/libcrypto.lib)
  set(OPENSSL_SSL_LIBRARY ${OPENSSL_ROOT_DIR}/lib/libssl.lib)
endif()

set(CMAKE_C_FLAGS_DEBUG "-DDEBUG -g -march=native -std=gnu17")
set(CMAKE_C_FLAGS_MINSIZEREL "-Os -march=native -std=gnu17")
set(CMAKE_C_FLAGS_RELWITHDEBINFO "-Os -Ofast -g -march=native -std=gnu17")
set(CMAKE_C_FLAGS_RELEASE "-Os -Ofast -march=native -std=gnu17")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS_RELWITHDEBUGINFO}")

set(BIN_DIRECTORY bin)

# Provider module
add_library(vigenere MODULE vigenere.c)
set_target_properties(vigenere PROPERTIES PREFIX "" OUTPUT_NAME "vigenere")
target_link_libraries(vigenere ${OPENSSL_CRYPTO_LIBRARY})

# Testing
add_executable(test_vigenere test_vigenere.c)
target_link_libraries(test_vigenere ${OPENSSL_CRYPTO_LIBRARY})
add_test(NAME vigenere COMMAND test_vigenere)
set_tests_properties(vigenere
  PROPERTIES ENVIRONMENT "OPENSSL_MODULES=${CMAKE_BINARY_DIR}"
  )

set(BINARY_TESTS_TARGETS
        test_vigenere
        )

# No installation instruction, as this should never be used in production
# If you still want to do so, just copy vigenere.so / vigenere.dll to an
# appropriate location.
