cmake_minimum_required(VERSION 2.8.9)

# Define project data
set (APPLICATION_VENDOR_NAME "freetuxtv")
set (APPLICATION_VENDOR_DOMAIN "https://github.com/freetuxtv/freetuxtv")
set (APPLICATION_NAME "FreetuxTV")
set (APPLICATION_VERSION_MAJOR 0)
set (APPLICATION_VERSION_MINOR 7)
set (APPLICATION_VERSION_PATCH 0)
if(NOT DEFINED APPLICATION_VERSION_BUILD)
    set(APPLICATION_VERSION_BUILD "")
endif()
if(CMAKE_BUILD_TYPE MATCHES Debug)
    set(APPLICATION_VERSION_BUILD ${APPLICATION_VERSION_BUILD}dbg)
endif()
set (APPLICATION_VERSION ${APPLICATION_VERSION_MAJOR}.${APPLICATION_VERSION_MINOR}.${APPLICATION_VERSION_PATCH}${APPLICATION_VERSION_BUILD})
set (APPLICATION_CONTACTMAIL "admin@freetuxtv.net")
set (APPLICATION_CONTACT "Contact FreetuxTV <${APPLICATION_CONTACTMAIL}>")
set (APPLICATION_PACKAGE_VENDOR "freetuxtv")
set (APPLICATION_PACKAGE_NAME "freetuxtv")
set (APPLICATION_EXE_NAME freetuxtv)
set (APPLICATION_BUGREPORT "https://github.com/freetuxtv/freetuxtv/issues")

project (freetuxtv C)

if(WIN32)
    set(AUTO_NEWLINE_STYLE WIN32)
else()
    set(AUTO_NEWLINE_STYLE UNIX)
endif()
if(CMAKE_HOST_SYSTEM MATCHES Windows)
    set(HOST_WIN32 TRUE)
else()
    set(HOST_WIN32 FALSE)
endif()

# Define install paths
if(WIN32)
    set(INSTALL_PATH_BIN ".")
    set(INSTALL_PATH_SHARE ".")
    set(INSTALL_PATH_DOC ".")
    set(INSTALL_PATH_APPLICATIONS ".")
    set(INSTALL_PATH_ICONS ".")
elseif(APPLE)
    set(INSTALL_PATH_BIN ".")
    set(INSTALL_PATH_SHARE ".")
    set(INSTALL_PATH_FRAMEWORKS "../Frameworks")
    set(INSTALL_PATH_DOC ".")
    set(INSTALL_PATH_APPLICATIONS ".")
    set(INSTALL_PATH_ICONS ".")
else(WIN32)
    set(CMAKE_INSTALL_PREFIX "/usr")
    set(INSTALL_PATH_BIN "${CMAKE_INSTALL_PREFIX}/bin/")
    set(INSTALL_PATH_SHARE "${CMAKE_INSTALL_PREFIX}/share/${APPLICATION_PACKAGE_NAME}")
    set(INSTALL_PATH_DOC "${CMAKE_INSTALL_PREFIX}/share/${APPLICATION_PACKAGE_NAME}/doc")
    set(INSTALL_PATH_APPLICATIONS "${CMAKE_INSTALL_PREFIX}/share/applications")
    set(INSTALL_PATH_ICONS "${CMAKE_INSTALL_PREFIX}/share/icons/")
    set(INSTALL_PATH_LOCALE "${CMAKE_INSTALL_PREFIX}/share/locale/")
endif(WIN32)

####################
# Macro definition
####################

macro(JOINLIST list_name sep output)
    # create empty list - necessary?
    # prefix and suffix elements
    foreach(l ${${list_name}})
        if(NOT DEFINED list_name_TMP)
            SET(list_name_TMP ${l})
        else()
            SET(list_name_TMP ${list_name_TMP}${sep}${l})
        endif()
    endforeach()
    # replace list by tmp list
    SET(${output} ${list_name_TMP})
    UNSET(list_name_TMP)
endmacro(JOINLIST)

####################
# Library checking
####################

# Check for pkg-config
find_package(PkgConfig REQUIRED)

# Check for GTK 3
pkg_check_modules(GTK3 REQUIRED gtk+-3.0)
if(GTK3_FOUND)
    message(STATUS "Found gtk3 ${GTK3_VERSION}")
    include_directories(${GTK3_INCLUDE_DIRS})
    link_directories(${GTK3_LIBRARY_DIRS})
    add_definitions(${GTK3_CFLAGS_OTHER} -DGTK_API_VERSION=3)
    list(APPEND PACKAGE_DEPS "libgtk-3-0 (>= ${GTK3_VERSION})")
endif()

# Check for libnotify
pkg_check_modules(LIBNOTIFY libnotify>=0.7.6 REQUIRED)
if(LIBNOTIFY_FOUND)
    message(STATUS "Found libnotify ${LIBNOTIFY_VERSION}")
    include_directories(${LIBNOTIFY_INCLUDE_DIRS})
    link_directories(${LIBNOTIFY_LIBRARY_DIRS})
    string(REPLACE "." ";" LIBNOTIFY_VERSION_LIST ${LIBNOTIFY_VERSION})
    list(GET LIBNOTIFY_VERSION_LIST 0 LIBNOTIFY_VERSION_MAJOR)
    list(GET LIBNOTIFY_VERSION_LIST 1 LIBNOTIFY_VERSION_MINOR)
    list(GET LIBNOTIFY_VERSION_LIST 2 LIBNOTIFY_VERSION_REVISION)
    add_definitions(-DLIBNOTIFY_VERSION_MAJOR=${LIBNOTIFY_VERSION_MAJOR} -DLIBNOTIFY_VERSION_MINOR=${LIBNOTIFY_VERSION_MINOR} -DLIBNOTIFY_VERSION_REVISION=${LIBNOTIFY_VERSION_REVISION})
    list(APPEND PACKAGE_DEPS "libnotify4 (>= ${LIBNOTIFY_VERSION})")
endif()

# Check for libvlc
pkg_check_modules(LIBVLC REQUIRED libvlc)
if(LIBVLC_FOUND)
    message(STATUS "Found libvlc ${LIBVLC_VERSION}")
    include_directories(${LIBVLC_INCLUDE_DIRS})
    link_directories(${LIBVLC_LIBRARY_DIRS})
    string(REPLACE "." ";" LIBVLC_VERSION_LIST ${LIBVLC_VERSION})
    list(GET LIBVLC_VERSION_LIST 0 LIBVLC_VERSION_MAJOR)
    list(GET LIBVLC_VERSION_LIST 1 LIBVLC_VERSION_MINOR)
    list(GET LIBVLC_VERSION_LIST 2 LIBVLC_VERSION_REVISION)
    add_definitions(-DLIBVLC_VERSION_MAJOR=${LIBVLC_VERSION_MAJOR} -DLIBVLC_VERSION_MINOR=${LIBVLC_VERSION_MINOR} -DLIBVLC_VERSION_REVISION=${LIBVLC_VERSION_REVISION})
    list(APPEND PACKAGE_DEPS "libvlc5 (>= ${LIBVLC_VERSION})")
endif()

# Check for sqlite3
pkg_check_modules(SQLITE3 REQUIRED sqlite3)
if(SQLITE3_FOUND)
    message(STATUS "Found sqlite3 ${SQLITE3_VERSION}")
    include_directories(${SQLITE3_INCLUDE_DIRS})
    link_directories(${SQLITE3_LIBRARY_DIRS})
    list(APPEND PACKAGE_DEPS "libsqlite3-0 (>= ${SQLITE3_VERSION})")
endif()

# Check for curl
pkg_check_modules(CURL REQUIRED libcurl)
if(CURL_FOUND)
    message(STATUS "Found curl ${CURL_VERSION}")
    include_directories(${CURL_INCLUDE_DIRS})
    link_directories(${CURL_LIBRARY_DIRS})
    list(APPEND PACKAGE_DEPS "libcurl4 (>= ${CURL_VERSION})")
endif()

# Setting up Intl
find_package (Intl REQUIRED)
find_package(Gettext REQUIRED)
include_directories(${INTL_INCLUDE_DIRS})
link_directories(${INTL_LIBRARY_DIRS})
add_definitions(-DENABLE_NLS -DGETTEXT_PACKAGE=\"${APPLICATION_PACKAGE_NAME}\")


####################
# Config file
####################

set(PACKAGE_LOCALE_DIR "${CMAKE_INSTALL_PREFIX}/share/locale")
set(PACKAGE_DATA_DIR "${CMAKE_INSTALL_PREFIX}/share")

# Configure a header file to pass some of the CMake settings
# to the source code
configure_file (
        "${PROJECT_SOURCE_DIR}/config.h.in"
        "${PROJECT_BINARY_DIR}/config.h"
)
add_definitions(-DHAVE_CONFIG_H)
configure_file (
        "${PROJECT_SOURCE_DIR}/version.h.in"
        "${PROJECT_BINARY_DIR}/version.h"
)
add_definitions(-DHAVE_VERSION_H)
include_directories (${PROJECT_BINARY_DIR})

####################
# Add sources
####################

include_directories (${PROJECT_SOURCE_DIR}/lib)
include_directories (${PROJECT_SOURCE_DIR}/src)

# Compatiblity with autotools
add_definitions(-DPACKAGE_LOCALE_DIR="${PACKAGE_LOCALE_DIR}")
add_definitions(-DPACKAGE_DATA_DIR="${PACKAGE_DATA_DIR}")
add_definitions(-DVERSION="${APPLICATION_VERSION}")
add_definitions(-DPACKAGE="${APPLICATION_PACKAGE_NAME}")
add_definitions(-DPACKAGE_BUGREPORT="${APPLICATION_BUGREPORT}")

set(LIBDBEVOLUTION_SRC
    lib/libdbevolution/db-evolution-instance.h
    lib/libdbevolution/db-evolution-instance.c
)

set(LIBVLCGTK_SRC
    lib/libvlc-gtk/gtk-libvlc-include.h
    lib/libvlc-gtk/gtk-libvlc-instance.h
    lib/libvlc-gtk/gtk-libvlc-instance.c
    lib/libvlc-gtk/gtk-libvlc-media.h
    lib/libvlc-gtk/gtk-libvlc-media.c
    lib/libvlc-gtk/gtk-libvlc-media-player.h
    lib/libvlc-gtk/gtk-libvlc-media-player.c
    lib/libvlc-gtk/gtk-libvlc-private.h
)

set(FREETUXTV_LIB_SRC
    src/gtk-builder-dialog.c
    src/gtk-builder-dialog.h
    src/gtk-builder-object.c
    src/gtk-builder-object.h
    src/gtk-builder-widget.c
    src/gtk-builder-widget.h
    src/gtk-builder-window.c
    src/gtk-builder-window.h
    src/gtk-date-time-picker.c
    src/gtk-date-time-picker.h
    src/gtk-progress-dialog.c
    src/gtk-progress-dialog.h
    src/lib-gmmkeys.c
    src/lib-gmmkeys.h
    src/lib-m3uparser.c
    src/lib-m3uparser.h
)

set(FREETUXTV_SRC
    src/freetuxtv-app.h
    src/freetuxtv-cclosure-marshal.c
    src/freetuxtv-cclosure-marshal.h
    src/freetuxtv-cellrenderer-channelslist.c
    src/freetuxtv-cellrenderer-channelslist.h
    src/freetuxtv-cellrenderer-recordingslist.c
    src/freetuxtv-cellrenderer-recordingslist.h
    src/freetuxtv-channel-infos.c
    src/freetuxtv-channel-infos.h
    src/freetuxtv-channels-group-infos.c
    src/freetuxtv-channels-group-infos.h
    src/freetuxtv-channels-list.c
    src/freetuxtv-channels-list.h
    src/freetuxtv-db-sync.c
    src/freetuxtv-db-sync.h
    src/freetuxtv-fileutils.c
    src/freetuxtv-fileutils.h
    src/freetuxtv-gladexml.h
    src/freetuxtv-i18n.h
    src/freetuxtv-models.c
    src/freetuxtv-models.h
    src/freetuxtv-player-error-dialog.c
    src/freetuxtv-player-error-dialog.h
    src/freetuxtv-recording-infos.c
    src/freetuxtv-recording-infos.h
    src/freetuxtv-recordings-list.c
    src/freetuxtv-recordings-list.h
    src/freetuxtv-tv-channel-infos.c
    src/freetuxtv-tv-channel-infos.h
    src/freetuxtv-tv-channels-list.c
    src/freetuxtv-tv-channels-list.h
    src/freetuxtv-utils.c
    src/freetuxtv-utils.h
    src/freetuxtv-window-add-channels-group.c
    src/freetuxtv-window-add-channels-group.h
    src/freetuxtv-window-add-recording.c
    src/freetuxtv-window-add-recording.h
    src/freetuxtv-window-channel-properties.c
    src/freetuxtv-window-channel-properties.h
    src/freetuxtv-window-main.c
    src/freetuxtv-window-main.h
    src/main.c
)

####################
# Translations
####################

find_program(GETTEXT_MSGFMT_EXECUTABLE msgfmt)

if(NOT GETTEXT_MSGFMT_EXECUTABLE)
    message(FATAL "msgfmt not found")
else(NOT GETTEXT_MSGFMT_EXECUTABLE)

    SET(catalogname ${APPLICATION_PACKAGE_NAME})

    FILE(GLOB PO_FILES po/*.po)
    SET(GMO_FILES)

    FOREACH(_poFile ${PO_FILES})
        GET_FILENAME_COMPONENT(_poFileName ${_poFile} NAME)
        STRING(REGEX REPLACE "^${catalogname}_?" "" _langCode ${_poFileName} )
        STRING(REGEX REPLACE "\\.po$" "" _langCode ${_langCode} )

        IF( _langCode )
            GET_FILENAME_COMPONENT(_lang ${_poFile} NAME_WE)
            SET(_gmoFile ${CMAKE_CURRENT_BINARY_DIR}/po/${_lang}.gmo)

            ADD_CUSTOM_COMMAND(OUTPUT ${_gmoFile}
                    COMMAND mkdir -p ${CMAKE_CURRENT_BINARY_DIR}/po
                    COMMAND ${GETTEXT_MSGFMT_EXECUTABLE} --check -o ${_gmoFile} ${_poFile}
                    DEPENDS ${_poFile})
            INSTALL(FILES ${_gmoFile} DESTINATION ${INSTALL_PATH_LOCALE}/${_langCode}/LC_MESSAGES/ RENAME ${catalogname}.mo)
            LIST(APPEND GMO_FILES ${_gmoFile})
        ENDIF( _langCode )

    ENDFOREACH(_poFile ${PO_FILES})

    ADD_CUSTOM_TARGET(translations ALL DEPENDS ${GMO_FILES})

ENDIF(NOT GETTEXT_MSGFMT_EXECUTABLE)

find_program(INTLTOOL_MERGE_EXECUTABLE intltool-merge)
if(NOT INTLTOOL_MERGE_EXECUTABLE)
    message(FATAL "intltool-merge not found")
else(NOT INTLTOOL_MERGE_EXECUTABLE)
    set(DESKTOP_FILE_IN ${CMAKE_CURRENT_SOURCE_DIR}/data/menu/freetuxtv.desktop.in)
    set(DESKTOP_FILE ${CMAKE_CURRENT_BINARY_DIR}/data/menu/freetuxtv.desktop)
    ADD_CUSTOM_COMMAND(
            OUTPUT ${DESKTOP_FILE}
            COMMAND mkdir -p ${CMAKE_CURRENT_BINARY_DIR}/data/menu/
            COMMAND ${INTLTOOL_MERGE_EXECUTABLE} -d ${CMAKE_CURRENT_SOURCE_DIR}/po/ ${DESKTOP_FILE_IN} ${DESKTOP_FILE}
            DEPENDS ${DESKTOP_FILE_IN})

    ADD_CUSTOM_TARGET(desktop_file ALL DEPENDS ${DESKTOP_FILE})
endif()

####################
# Create Executable
####################

add_executable(${APPLICATION_EXE_NAME} ${LIBDBEVOLUTION_SRC} ${LIBVLCGTK_SRC} ${FREETUXTV_LIB_SRC} ${FREETUXTV_SRC})
target_link_libraries(${APPLICATION_EXE_NAME} ${GTK3_LIBRARIES} ${LIBNOTIFY_LIBRARIES} ${LIBVLC_LIBRARIES} ${SQLITE3_LIBRARIES} ${CURL_LIBRARIES})
install(TARGETS ${APPLICATION_EXE_NAME} DESTINATION ${INSTALL_PATH_BIN})

############################
# Common files installation
############################

# Documentation file
set(DOCFILES_EXT "")
if(WIN32)
    set(DOCFILES_EXT ".txt")
endif(WIN32)

# README file
configure_file (
        "${PROJECT_SOURCE_DIR}/README"
        "${PROJECT_BINARY_DIR}/README${DOCFILES_EXT}"
        NEWLINE_STYLE ${AUTO_NEWLINE_STYLE}
)
install(FILES "${PROJECT_BINARY_DIR}/README${DOCFILES_EXT}" DESTINATION ${INSTALL_PATH_DOC})

# COPYING file
configure_file (
        "${PROJECT_SOURCE_DIR}/COPYING"
        "${PROJECT_BINARY_DIR}/COPYING${DOCFILES_EXT}"
        NEWLINE_STYLE ${AUTO_NEWLINE_STYLE}
)
install(FILES "${PROJECT_BINARY_DIR}/COPYING${DOCFILES_EXT}" DESTINATION ${INSTALL_PATH_DOC})

# AUTHORS file
configure_file (
        "${PROJECT_SOURCE_DIR}/AUTHORS"
        "${PROJECT_BINARY_DIR}/AUTHORS${DOCFILES_EXT}"
        NEWLINE_STYLE ${AUTO_NEWLINE_STYLE}
)
install(FILES "${PROJECT_BINARY_DIR}/AUTHORS${DOCFILES_EXT}" DESTINATION ${INSTALL_PATH_DOC})

# ChangeLog file
configure_file (
        "${PROJECT_SOURCE_DIR}/ChangeLog"
        "${PROJECT_BINARY_DIR}/ChangeLog${DOCFILES_EXT}"
        NEWLINE_STYLE ${AUTO_NEWLINE_STYLE}
)
install(FILES "${PROJECT_BINARY_DIR}/ChangeLog${DOCFILES_EXT}" DESTINATION ${INSTALL_PATH_DOC})

# INSTALL file
configure_file (
        "${PROJECT_SOURCE_DIR}/INSTALL"
        "${PROJECT_BINARY_DIR}/INSTALL${DOCFILES_EXT}"
        NEWLINE_STYLE ${AUTO_NEWLINE_STYLE}
)
install(FILES "${PROJECT_BINARY_DIR}/INSTALL${DOCFILES_EXT}" DESTINATION ${INSTALL_PATH_DOC})

###############################
# Resources files installation
###############################

# Desktop file
if(UNIX AND NOT WIN32)
    install(FILES ${CMAKE_CURRENT_BINARY_DIR}/data/menu/freetuxtv.desktop DESTINATION ${INSTALL_PATH_APPLICATIONS})
endif(UNIX AND NOT WIN32)

# Splashscreen
install(FILES "${PROJECT_SOURCE_DIR}/data/splashscreen.png" DESTINATION ${INSTALL_PATH_SHARE})

# Database file
install(FILES "${PROJECT_SOURCE_DIR}/data/sqlite3-create-tables.sql" DESTINATION ${INSTALL_PATH_SHARE})

# UI files
install(DIRECTORY "${PROJECT_SOURCE_DIR}/data/ui/" DESTINATION ${INSTALL_PATH_SHARE}/ui FILES_MATCHING PATTERN "*.glade")

# Data files
install(FILES "${PROJECT_SOURCE_DIR}/data/channels_groups.xml" DESTINATION ${INSTALL_PATH_SHARE})
install(FILES "${PROJECT_SOURCE_DIR}/data/tv_channels.xml" DESTINATION ${INSTALL_PATH_SHARE})

# Icons
install(FILES "${PROJECT_SOURCE_DIR}/data/icons/hicolor_apps_16x16_freetuxtv.png" DESTINATION ${INSTALL_PATH_ICONS}/hicolor/16x16/apps/ RENAME freetuxtv.png)
install(FILES "${PROJECT_SOURCE_DIR}/data/icons/hicolor_apps_22x22_freetuxtv.png" DESTINATION ${INSTALL_PATH_ICONS}/hicolor/22x22/apps/ RENAME freetuxtv.png)
install(FILES "${PROJECT_SOURCE_DIR}/data/icons/hicolor_apps_32x32_freetuxtv.png" DESTINATION ${INSTALL_PATH_ICONS}/hicolor/32x32/apps/ RENAME freetuxtv.png)
install(FILES "${PROJECT_SOURCE_DIR}/data/icons/hicolor_apps_48x48_freetuxtv.png" DESTINATION ${INSTALL_PATH_ICONS}/hicolor/48x48/apps/ RENAME freetuxtv.png)
install(FILES "${PROJECT_SOURCE_DIR}/data/icons/hicolor_apps_64x64_freetuxtv.png" DESTINATION ${INSTALL_PATH_ICONS}/hicolor/64x64/apps/ RENAME freetuxtv.png)
install(FILES "${PROJECT_SOURCE_DIR}/data/icons/hicolor_apps_128x128_freetuxtv.png" DESTINATION ${INSTALL_PATH_ICONS}/hicolor/128x128/apps/ RENAME freetuxtv.png)
install(FILES "${PROJECT_SOURCE_DIR}/data/icons/hicolor_apps_scalable_freetuxtv.svg" DESTINATION ${INSTALL_PATH_ICONS}/hicolor/scalable/apps/ RENAME freetuxtv.svg)

#################
# Build package
#################

INCLUDE(InstallRequiredSystemLibraries)

# CPack confile file
configure_file(${PROJECT_SOURCE_DIR}/CPackOptions.cmake.in
        ${PROJECT_BINARY_DIR}/CPackOptions.cmake
        @ONLY)
set(CPACK_PROJECT_CONFIG_FILE ${CMAKE_BINARY_DIR}/CPackOptions.cmake)

if(WIN32)
    if(CMAKE_BUILD_TYPE MATCHES Debug)
        SET(CPACK_GENERATOR ZIP)
    else()
        SET(CPACK_GENERATOR ZIP NSIS)
    endif()
elseif(APPLE)
    configure_file(${PROJECT_SOURCE_DIR}/platforms/osx/Info.plist.in
            ${PROJECT_BINARY_DIR}/platforms/osx/Info.plist
            @ONLY)
    configure_file(${PROJECT_SOURCE_DIR}/platforms/osx/run.sh.in
            ${PROJECT_BINARY_DIR}/platforms/osx/${APPLICATION_NAME}
            @ONLY)
    SET(CPACK_GENERATOR Bundle)
else()
    SET(CPACK_GENERATOR TGZ DEB)
endif(WIN32)

SET(CPACK_PACKAGE_NAME ${APPLICATION_PACKAGE_NAME})
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY ${APPLICATION_DESCRIPTION_SUMMARY})
SET(CPACK_PACKAGE_VENDOR ${APPLICATION_VENDOR_NAME})
SET(CPACK_PACKAGE_CONTACT ${APPLICATION_CONTACT})
SET(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README${DOCFILES_EXT}")
SET(CPACK_PACKAGE_VERSION_MAJOR ${APPLICATION_VERSION_MAJOR})
SET(CPACK_PACKAGE_VERSION_MINOR ${APPLICATION_VERSION_MINOR})
SET(CPACK_PACKAGE_VERSION_PATCH ${APPLICATION_VERSION_PATCH}${APPLICATION_VERSION_BUILD})
SET(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/COPYING${DOCFILES_EXT}")
SET(CPACK_PACKAGE_INSTALL_DIRECTORY ${APPLICATION_PACKAGE_NAME}) # Useful for registry key and NSIS auto uninstaller

if(CMAKE_BUILD_TYPE MATCHES Debug)
    SET(CPACK_STRIP_FILES FALSE)
    SET(CPACK_SOURCE_STRIP_FILES FALSE)
else()
    SET(CPACK_STRIP_FILES TRUE)
    SET(CPACK_SOURCE_STRIP_FILES TRUE)
endif()

if(WIN32)
    # NSIS
    SET(CPACK_NSIS_DISPLAY_NAME "${APPLICATION_NAME}")
elseif(APPLE)
    if (NOT CPACK_SYSTEM_NAME)
        IF(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
            SET(CPACK_SYSTEM_NAME amd64)
        ELSEIF(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "i686")
            SET(CPACK_SYSTEM_NAME i386)
        ENDIF(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
    endif()

    SET(CPACK_BUNDLE_NAME "${APPLICATION_NAME}")
else()
    if (NOT CPACK_SYSTEM_NAME)
        IF(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
            SET(CPACK_SYSTEM_NAME amd64)
        ELSEIF(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "i686")
            SET(CPACK_SYSTEM_NAME i386)
        ENDIF(${CMAKE_SYSTEM_PROCESSOR} STREQUAL "x86_64")
    endif()

    JOINLIST(PACKAGE_DEPS ", " DEBIAN_DEPS)

    SET(CPACK_DEBIAN_PACKAGE_NAME ${APPLICATION_PACKAGE_NAME})
    SET(CPACK_DEBIAN_PACKAGE_VERSION ${APPLICATION_VERSION})
    SET(CPACK_DEBIAN_PACKAGE_DEPENDS ${DEBIAN_DEPS})
    SET(CPACK_DEBIAN_PACKAGE_SECTION "misc")
    # SET(CPACK_DEBIAN_PACKAGE_ARCHITECTURE ${DEB_ARCH})
ENDIF()
SET(CPACK_PACKAGE_EXECUTABLES "${APPLICATION_EXE_NAME}" "${APPLICATION_NAME}")
include(CPack)
