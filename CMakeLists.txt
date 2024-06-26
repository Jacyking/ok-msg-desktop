cmake_minimum_required(VERSION 3.21)

if(POLICY CMP0091)
  cmake_policy(SET CMP0091 NEW)
endif(POLICY CMP0091)

if(POLICY CMP0135)
  cmake_policy(SET CMP0135 NEW)
endif(POLICY CMP0135)
include(FetchContent)

project(OkMSG-Project)

get_filename_component(third_party_loc "3rdparty" REALPATH)
get_filename_component(submodules_loc "src" REALPATH)
get_filename_component(cmake_helpers_loc "cmake" REALPATH)

include(cmake/init_target.cmake)
include(cmake/generate_target.cmake)
include(cmake/generate_appdata_changelog.cmake)
include(cmake/dependencies.cmake)
include(cmake/build.cmake)
include(cmake/develop.cmake)
include(cmake/platform.cmake)

include_directories(.)
include_directories(${PROJECT_SOURCE_DIR}/3rdparty/libvpx)

add_subdirectory(src)

# Add RC files.
set(${PROJECT_NAME}_RC_FILES src/icon/icon.rc)

# Add QRC files.
set(${PROJECT_NAME}_QRC_FILES UI/resources/resources.qrc)

set(${PROJECT_NAME}_RESOURCES ${${PROJECT_NAME}_RC_FILES}
        ${${PROJECT_NAME}_QRC_FILES})

set(${PROJECT_NAME}_SOURCES src/main.cpp)

qt5_add_translation(${PROJECT_NAME}_QM_FILES translations/zh_CN.ts)

file(
        WRITE "${PROJECT_BINARY_DIR}/translations.qrc.in"
        "<!DOCTYPE RCC>
<RCC version=\"1.0\">
  <qresource prefix=\"/translations/${PROJECT_NAME}\">
")

foreach(qm ${${PROJECT_NAME}_QM_FILES})
  get_filename_component(qm_name ${qm} NAME)
  file(APPEND "${PROJECT_BINARY_DIR}/translations.qrc.in"
          "    <file alias=\"${qm_name}\">${qm}</file>\n")
endforeach(qm)

file(APPEND "${PROJECT_BINARY_DIR}/translations.qrc.in" "  </qresource>
</RCC>")

execute_process(
        COMMAND
        ${CMAKE_COMMAND} -E copy_if_different
        ${PROJECT_BINARY_DIR}/translations.qrc.in
        ${PROJECT_BINARY_DIR}/translations.qrc)

qt5_add_resources(${PROJECT_NAME}_RESOURCES
        ${PROJECT_BINARY_DIR}/translations.qrc)

if(ENABLE_PLUGINS)
  include(${PROJECT_SOURCE_DIR}/plugins/plugins.cmake)
endif()

