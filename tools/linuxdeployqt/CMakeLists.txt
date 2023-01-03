cmake_minimum_required(VERSION 3.5)

project(linuxdeployqt VERSION 0.1 LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# expose version data as compiler definition
add_definitions("-DLINUXDEPLOYQT_VERSION=\"${GIT_TAG_NAME}\"")
add_definitions("-DLINUXDEPLOYQT_GIT_COMMIT=\"${GIT_COMMIT}\"")
add_definitions("-DBUILD_DATE=\"${DATE}\"")
add_definitions("-DBUILD_NUMBER=\"${BUILD_NUMBER}\"")

find_package(Qt6 REQUIRED COMPONENTS Core)
find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets LinguistTools)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets LinguistTools)

# update excludelist
message(STATUS "Updating excludelist...")
execute_process(
    COMMAND bash ${CMAKE_CURRENT_SOURCE_DIR}/../generate-excludelist.sh
    OUTPUT_VARIABLE EXCLUDELIST
    TIMEOUT 10
    RESULT_VARIABLE EXCLUDELIST_RESULT
)
if(NOT EXCLUDELIST_RESULT EQUAL 0)
    message(WARNING "Updating excludelist failed, using outdated copy")
endif()
mark_as_advanced(EXCLUDELIST EXCLUDELIST_RESULT)


##
set(PROJECT_SOURCES
        main.cpp
        shared.h
        shared.cpp
        excludelist.h
)

qt_add_executable(linuxdeployqt
    MANUAL_FINALIZATION
    ${PROJECT_SOURCES}
)

target_link_libraries(linuxdeployqt PRIVATE Qt${QT_VERSION_MAJOR}::Widgets)


set_target_properties(linuxdeployqt PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER my.example.com
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

install(TARGETS linuxdeployqt
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

qt_finalize_executable(linuxdeployqt)
