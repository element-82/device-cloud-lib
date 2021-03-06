#
# Copyright (C) 2017-2018 Wind River Systems, Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at:
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software  distributed
# under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES
# OR CONDITIONS OF ANY KIND, either express or implied.
#

set( CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/build-sys/cmake/modules" ${CMAKE_MODULE_PATH} )
set( IOT_CONFIG_FILE "build.yml" )
include ( ReadYamlFile )
read_yaml_file( ${IOT_CONFIG_FILE} )

cmake_minimum_required( VERSION 2.8.5 )
project( ${IOT_PRODUCT} )

if( WIN32 )
	set( IOT_USER "SYSTEM" )
	set( IOT_GROUP "" )
endif()

# Generate Project Version information (from last git commit)
include( VersionSupport )
string( REGEX REPLACE "20([0-9][0-9])" "\\1" IOT_VERSION "${PROJECT_COMMIT_DATE}" )
string( REPLACE "-" "." IOT_VERSION "${IOT_VERSION}" )
version_split( IOT_VERSION "${IOT_VERSION}" )

# Project information
set( PROJECT_NAME                      "${CMAKE_PROJECT_NAME}" )
set( PROJECT_RESOURCE_PATH_WIN32       "${CMAKE_SOURCE_DIR}/build-sys/windows")

# Device manager
set( IOT_DEVICE_MANAGER_TARGET         "iot-device-manager" )
set( IOT_DEVICE_MANAGER_NAME           "${IOT_PRODUCT} Device Manager" )
set( IOT_DEVICE_MANAGER_DESCRIPTION    "Registers and handles device level requests for ${IOT_PRODUCT} devices."
                                       "Client that registers with the cloud to handle over-the-air updates,"
                                       "device reboot/shutdown, agent reset etc." )
string( REPLACE ";" " " IOT_DEVICE_MANAGER_DESCRIPTION "${IOT_DEVICE_MANAGER_DESCRIPTION}" )

# Options
include( DefineCompilerFlags )
include( GetFullPath )
include( GNUInstallDirs )
include( OptionSelect )
option_select( IOT_JSON_LIBRARY
	DESCRIPTION "JSON library to use"
	DEFAULT "jsmn" "jansson" "jsmn" "json-c"
)
option_select( IOT_MQTT_LIBRARY
	DESCRIPTION "MQTT library to use"
	DEFAULT "paho" "mosquitto" "paho"
)
option_select( IOT_WEBSOCKET_LIBRARY
	DESCRIPTION "Websocket library to use"
	DEFAULT "libwebsockets" "libwebsockets" "civetweb"
)
option_ensure_set( IOT_PLUGIN_SUPPORT   "allow dynamic plug-in support" ON )
option_ensure_set( IOT_STACK_ONLY       "build library without the use of the heap" OFF )
option_ensure_set( IOT_THREAD_SUPPORT   "support the use of threads" ON )

# Enforce Build Type
# set a default build type if none was specified
set( DEFAULT_BUILD_TYPE "Debug" )
if( NOT CMAKE_CONFIGURATION_TYPES )
	if ( NOT CMAKE_BUILD_TYPE )
		message( STATUS "Setting build type to '${DEFAULT_BUILD_TYPE}' as none was specified." )
	endif ( NOT CMAKE_BUILD_TYPE )
	option_select( CMAKE_BUILD_TYPE DEFAULT ${DEFAULT_BUILD_TYPE} DESCRIPTION "Type of build"
		Coverage Debug MinSizeRel Release RelWithDebInfo )
	message( STATUS "Selected build type is '${CMAKE_BUILD_TYPE}'" )
endif( NOT CMAKE_CONFIGURATION_TYPES )

# Base build configuration file
set( IOT_IN_FILE  "iot_build.h.in" )
set( IOT_OUT_FILE "iot_build.h" )
set( IOT_GIT_SHA  "${PROJECT_GIT_SHA}" )
configure_file( "${IOT_IN_FILE}" "${CMAKE_BINARY_DIR}/${IOT_OUT_FILE}" @ONLY )

# Setup Project output directories
set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY    "${CMAKE_BINARY_DIR}/lib" )
set( CMAKE_LIBRARY_OUTPUT_DIRECTORY    "${CMAKE_BINARY_DIR}/lib" )
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY    "${CMAKE_BINARY_DIR}/bin" )
# Location to find public include files when building sample applications
set( CMAKE_INCLUDE_OUTPUT_DIRECTORY    "${CMAKE_SOURCE_DIR}/src/api/public" )

# Installation directories
set( INSTALL_BIN_DIR         "${CMAKE_INSTALL_BINDIR}" )
set( INSTALL_CONFIG_DIR      "${CMAKE_INSTALL_SYSCONFDIR}" )
set( INSTALL_EXAMPLES_DIR    "${CMAKE_INSTALL_DATAROOTDIR}/${IOT_LIBRARY_NAME}" )
set( INSTALL_INCLUDE_DIR     "${CMAKE_INSTALL_INCLUDEDIR}" )
set( INSTALL_LIB_DIR         "${CMAKE_INSTALL_LIBDIR}" )
set( INSTALL_PLUGIN_DIR      "${CMAKE_INSTALL_LIBDIR}" )
set( INSTALL_VAR_DIR         "${CMAKE_INSTALL_LOCALSTATEDIR}" )

if ( CPACK_GENERATOR STREQUAL "DEB" OR
	CPACK_GENERATOR STREQUAL "RPM" )
	set( INSTALL_CONFIG_DIR      "${IOT_DEFAULT_DIR_CONFIG}" )
	set( INSTALL_VAR_DIR         "${IOT_DEFAULT_DIR_RUNTIME}" )
endif ( CPACK_GENERATOR STREQUAL "DEB" OR
	CPACK_GENERATOR STREQUAL "RPM" )

# 3rd party libraries
find_package( Threads )
set( PAHO_PREFER_STATIC   ON )
set( PAHO_USE_SSL         ON )
set( PAHO_ASYNC           ${IOT_THREAD_SUPPORT} )
if ( IOT_JSON_LIBRARY STREQUAL "jansson" )
	find_package( Jansson REQUIRED )
	add_definitions( "-DIOT_JSON_JANSSON" )
	set( JSON_INCLUDE_DIR "${JANSSON_INCLUDE_DIR}" )
	set( JSON_LIBRARIES   "${JANSSON_LIBRARIES}" )
elseif ( IOT_JSON_LIBRARY STREQUAL "json-c" )
	find_package( JsonC REQUIRED )
	add_definitions( "-DIOT_JSON_JSONC" )
	set( JSON_INCLUDE_DIR "${JSONC_INCLUDE_DIR}" )
	set( JSON_LIBRARIES   "${JSONC_LIBRARIES}" )
else()
	find_package( Jsmn REQUIRED )
	set( JSON_INCLUDE_DIR "${JSMN_INCLUDE_DIR}" )
	set( JSON_LIBRARIES   "${JSMN_LIBRARIES}" )
	set( JSON_DEFINES     "${JSMN_DEFINES}" )
endif()

if ( IOT_MQTT_LIBRARY STREQUAL "mosquitto" )
	find_package( Mosquitto REQUIRED )
	add_definitions( "-DIOT_MQTT_MOSQUITTO" )
	set( MQTT_INCLUDE_DIR "${MOSQUITTO_INCLUDE_DIR}" )
	set( MQTT_LIBRARIES "${MOSQUITTO_LIBRARIES}" )
	set( MQTT_SSL_SUPPORT ON )
else()
	find_package( Paho REQUIRED )
	set( MQTT_INCLUDE_DIR "${PAHO_INCLUDE_DIR}" )
	set( MQTT_LIBRARIES "${PAHO_LIBRARIES}" )
	set( MQTT_SSL_SUPPORT ${PAHO_SSL_SUPPORT} )
endif()

if ( IOT_WEBSOCKET_LIBRARY STREQUAL "civetweb" )
	find_package( Civetweb REQUIRED )
	add_definitions( "-DIOT_WEBSOCKET_CIVETWEB" ${CIVETWEB_DEFINITIONS} )
	set( WEBSOCKET_INCLUDE_DIR "${CIVETWEB_INCLUDE_DIR}" )
	set( WEBSOCKET_LIBRARIES "${CIVETWEB_LIBRARIES}" )
else()
	find_package( LibWebSockets REQUIRED )
	set( WEBSOCKET_INCLUDE_DIR "${LIBWEBSOCKETS_INCLUDE_DIR}" )
	set( WEBSOCKET_LIBRARIES "${LIBWEBSOCKETS_LIBRARIES}" )
endif()

set( OSAL_PREFER_STATIC ON )
find_package( Osal REQUIRED )
add_definitions( ${OSAL_DEFINITIONS} )
include_directories( SYSTEM
	"${OSAL_INCLUDE_DIR}"
)

set( CMAKE_POSITION_INDEPENDENT_CODE ON )
set( LIB_OPTIONS "IOT_THREAD_SUPPORT" "IOT_STACK_ONLY" )
foreach( LIB_OPTION ${LIB_OPTIONS} )
	if ( ${LIB_OPTION} )
		add_definitions( "-D${LIB_OPTION}" )
	endif( ${LIB_OPTION} )
endforeach( LIB_OPTION )
include_directories( "src/api/public" "src" "${CMAKE_BINARY_DIR}" )
add_subdirectory( "src" )
add_subdirectory( "share" )
# Packaging support
add_subdirectory( "build-sys/cpack" )

include( TestSupport )
include( CPack )

