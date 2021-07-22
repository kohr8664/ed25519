cmake_minimum_required(VERSION 3.21.0)

# Setup the project
project(ed25519)

# Sources
set(
    SRCS
    ./src/add_scalar.c
    ./src/fe.c
    ./src/ge.c
    ./src/key_exchange.c
    ./src/keypair.c
    ./src/sc.c
    ./src/seed.c
    ./src/sha512.c
    ./src/sign.c
    ./src/verify.c
)

# Submodules
add_subdirectory("./extra/tinyBLAKE3")

# Global include directories
include_directories(
    ./include
    ./extra/tinyBLAKE3/include
)

# Global link libraries
link_libraries(
    blake3
)

# Add static library target
add_library(ed25519_static STATIC ${SRCS})
set_target_properties(
    ed25519_static
    PROPERTIES
        OUTPUT_NAME "ed25519"   
)

# Add dynamic library target
add_library(ed25519_dynamic SHARED ${SRCS})
set_target_properties(
    ed25519_dynamic
    PROPERTIES
        OUTPUT_NAME "ed25519"   
)

# Add test executable target
add_executable(ed25519_test ./src/test/test.c)
target_link_libraries(ed25519_test PRIVATE ed25519_static)