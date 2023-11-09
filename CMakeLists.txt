cmake_minimum_required(VERSION 3.14)

set(LIBRARY_NAME "flecs_systems_sokol")
set(LIBRARY_ALIAS "flecs_hub::systems::sokol")

project(${LIBRARY_NAME} VERSION 1.0.0 LANGUAGES C)

if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(FATAL_ERROR "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there.")
endif()

##################################
########### LIBRARY ##############
##################################

file(GLOB_RECURSE HEADERS CONFIGURE_DEPENDS
  "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h"
)
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c")

add_library(${LIBRARY_NAME}
  ${HEADERS}
  ${SOURCES}
  ${CMAKE_CURRENT_SOURCE_DIR}/src/sokol/sokol.m
)
add_library(${LIBRARY_ALIAS} ALIAS ${LIBRARY_NAME})
target_include_directories(${LIBRARY_NAME} PUBLIC ${PROJECT_SOURCE_DIR}/include)
set_target_properties(${LIBRARY_NAME} PROPERTIES C_STANDARD 99)

target_compile_definitions(
  # TARGET_PROPERTY:TYPE == STATIC_LIBRARY ? ${LIBRARY_NAME}_STATIC : ""
  ${LIBRARY_NAME} PUBLIC $<$<STREQUAL:$<TARGET_PROPERTY:${LIBRARY_NAME},TYPE>,STATIC_LIBRARY>:;${LIBRARY_NAME}_STATIC>
)

##################################
### FIND PACKAGE DEPENDENCIES ####
##################################

find_package(OpenGL REQUIRED OpenGL)
target_link_libraries(${LIBRARY_NAME} PRIVATE OpenGL::GL)

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  message(FATAL_ERROR "IMPLEMENT ME")
  #"lib": ["m", "GL", "X11", "Xi", "Xcursor", "dl"]
endif()

if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
  # unsupported or implement ldflags in project.json
  message(FATAL_ERROR "Unsupported platform")
endif()

##################################
######## CPM DEPENDENCIES ########
##################################

get_target_property(BUILD_LIBRARY_TYPE ${LIBRARY_NAME} TYPE)
if(BUILD_LIBRARY_TYPE STREQUAL "STATIC_LIBRARY")
  set(STR_IS_STATIC_LIBRARY "ON")
  set(STR_IS_SHARED_LIBRARY "OFF")
else()
  set(STR_IS_STATIC_LIBRARY "OFF")
  set(STR_IS_SHARED_LIBRARY "ON") 
endif()

include(cmake/get_cpm.cmake)

CPMAddPackage(
  VERSION 3.2.8
  GITHUB_REPOSITORY SanderMertens/flecs
  OPTIONS "FLECS_STATIC ${STR_IS_STATIC_LIBRARY}"
          "FLECS_SHARED ${STR_IS_SHARED_LIBRARY}"
)
target_link_libraries(${LIBRARY_NAME} PUBLIC
  # TARGET_PROPERTY:TYPE == STATIC_LIBRARY ? flecs::flecs_static : flecs::flecs
  $<IF:$<STREQUAL:$<TARGET_PROPERTY:${LIBRARY_NAME},TYPE>,STATIC_LIBRARY>,flecs::flecs_static,flecs::flecs>
)

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-gui#913e6849988f4b43ecbd85451289c4fc46b82094")
target_link_libraries(${LIBRARY_NAME} PUBLIC flecs_hub::components::gui)

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-input#2d60f0045741c588ba45cb3380a0b7a5a1a4732c")
target_link_libraries(${LIBRARY_NAME} PUBLIC flecs_hub::components::input)

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-graphics#4a8722f770ab1ea300935da9a7043f9d826870e6")
target_link_libraries(${LIBRARY_NAME} PUBLIC flecs_hub::components::graphics)

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-transform#83ebab3f8ad86a3f76cd10a8affcc335789af5b9")
target_link_libraries(${LIBRARY_NAME} PUBLIC flecs_hub::components::transform)

CPMAddPackage("gh:flecs-hub-cmake/flecs-components-geometry#ae45f5a2433804a9d1e6e62dad55b72a4618bcf4")
target_link_libraries(${LIBRARY_NAME} PUBLIC flecs_hub::components::geometry)

CPMAddPackage("gh:flecs-hub-cmake/flecs-systems-transform#14a7add4672a5c83adbea7819c8649872ba10d1e")
target_link_libraries(${LIBRARY_NAME} PUBLIC flecs_hub::systems::transform)

CPMAddPackage("gh:flecs-hub-cmake/flecs-game#af1f5dd4aca4ae93d25096e590c14b320a3ae38e")
target_link_libraries(${LIBRARY_NAME} PUBLIC flecs_hub::game)