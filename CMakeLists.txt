cmake_minimum_required(VERSION 3.11 FATAL_ERROR)

include(local_conf.cmake OPTIONAL)

include(FetchContent)
FetchContent_Declare(
	ccommon
	GIT_REPOSITORY https://github.com/EddyTheCo/Common.git
	GIT_TAG main
	)
FetchContent_MakeAvailable(ccommon)
version_from_git(
	LOG       OFF
	TIMESTAMP "%Y%m%d%H%M%S"
	)

project(lGPIOs VERSION ${VERSION} DESCRIPTION "Library to control the GPIOs " LANGUAGES C)

set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
	message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
	set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
		STRING "Choose the type of build." FORCE)
	# Set the possible values of build type for cmake-gui
	set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
		"Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

include(GNUInstallDirs)

add_library(lgpio
	src/lgCtx.c
	src/lgCtx.h
	src/lgDbg.c
	src/lgDbg.h
	src/lgErr.c
	src/lgGpio.c
	src/lgGpio.h
	src/lgHdl.c
	src/lgHdl.h
	src/lgI2C.c	
	src/lgNotify.c
	src/lgPthAlerts.c
	src/lgPthAlerts.h
	src/lgPthTx.c
	src/lgPthTx.h
	src/lgSerial.c
	src/lgSPI.c
	src/lgThread.c
	src/lgUtil.c
	)

target_include_directories(lgpio PUBLIC  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	"$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/lgpio>")

add_library(rgpio
	src/rgpio.c
	src/lgCfg.c
	src/lgCfg.h
	src/lgDbg.c
	src/lgDbg.h
	src/lgMD5.c
	src/lgMD5.h
	)
target_include_directories(rgpio PUBLIC  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
	"$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/lgpio>")
target_link_libraries(rgpio PUBLIC lgpio)

add_executable(rgpiod
	src/rgpiod.c
	src/rgpiod.h
	src/lgCmd.c
	src/lgCmd.h
	src/lgCfg.c
	src/lgCfg.h
	src/lgDbg.c
	src/lgDbg.h
	src/lgPthSocket.c
	src/lgExec.c	
	src/lgFile.c
	src/lgScript.c
	src/lgMD5.c
	src/lgMD5.h
	)
target_link_libraries(rgpiod PRIVATE lgpio)

add_executable(rgs
	src/rgs.c
	src/lgCmd.c
	src/lgCmd.h
	src/lgDbg.c
	src/lgDbg.h
	src/lgMD5.c
	src/lgMD5.h
	src/lgCfg.c
	src/lgCfg.h
	)
target_link_libraries(rgs PRIVATE lgpio)

target_compile_options(rgpio PRIVATE -O3 -Wall -pthread -fpic)


install(TARGETS lgpio rgpio rgs rgpiod EXPORT lgpioTargets
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} 
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	)
install(DIRECTORY include/
	DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/lgpio
	)

install(EXPORT lgpioTargets
	FILE lgpioTargets.cmake
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/lgpio
	)

include(CMakePackageConfigHelpers)
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
	"${CMAKE_CURRENT_BINARY_DIR}/lgpioConfig.cmake"
	INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/lgpio
	NO_SET_AND_CHECK_MACRO
	NO_CHECK_REQUIRED_COMPONENTS_MACRO
	)
write_basic_package_version_file(
	"${CMAKE_CURRENT_BINARY_DIR}/lgpioConfigVersion.cmake"
	VERSION ${VERSION}
	COMPATIBILITY SameMajorVersion
	)
install(FILES
	${CMAKE_CURRENT_BINARY_DIR}/lgpioConfig.cmake
	${CMAKE_CURRENT_BINARY_DIR}/lgpioConfigVersion.cmake
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/lgpio
	)
export(EXPORT lgpioTargets
	FILE "${CMAKE_CURRENT_BINARY_DIR}/lgpioTargets.cmake"
	)
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
	include(CTest)
	set(CPACK_PACKAGE_FILE_NAME "${PROJECT_NAME}-v${SEMVER}-${CMAKE_SYSTEM_NAME}_${CMAKE_SYSTEM_VERSION}-${CMAKE_SYSTEM_PROCESSOR}-${CMAKE_CXX_COMPILER_ID}")
	if(NOT BUILD_SHARED_LIBS)
		set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_FILE_NAME}-static")
	endif(NOT BUILD_SHARED_LIBS)
	include(CPack)
endif()
#if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND BUILD_TESTING)
#	add_subdirectory(tests)
#endif()
#if(BUILD_DOCS)
#endif()
