cmake_minimum_required(VERSION 4.0 FATAL_ERROR)
cmake_policy(VERSION 4.0)

project(0verkill C)

include(GNUInstallDirs)
include(CheckFunctionExists)
include(CheckIncludeFile)
include(CheckStructHasMember)
include(FindPkgConfig)

option(WITH_X "Also build X version." ON)
set(BASE_DIR "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_DATADIR}/0verkill" CACHE STRING "Data directory.")
message(STATUS "BASE_DIR: ${BASE_DIR}")

check_function_exists(gettimeofday HAVE_GETTIMEOFDAY)
check_function_exists(select HAVE_SELECT)
check_function_exists(strtol HAVE_STRTOL)
check_function_exists(strtoul HAVE_STRTOUL)
check_function_exists(getopt HAVE_GETOPT)
check_function_exists(psignal HAVE_PSIGNAL)
check_function_exists(access HAVE_ACCESS)

check_include_file(fcntl.h HAVE_FCNTL_H)
check_include_file(float.h HAVE_FLOAT_H)
check_include_file(inttypes.h HAVE_INTTYPES_H)
check_include_file(linux/kd.h HAVE_LINUX_KD_H)
check_include_file(linux/vt.h HAVE_LINUX_VT_H)
check_include_file(pthread.h HAVE_PTHREAD_H)
check_include_file(siginfo.h HAVE_SIGINFO_H)
check_include_file(stdint.h HAVE_STDINT_H)
check_include_file(stdio.h HAVE_STDIO_H)
check_include_file(stdlib.h HAVE_STDLIB_H)
check_include_file(strings.h HAVE_STRINGS_H)
check_include_file(string.h HAVE_STRING_H)
check_include_file(sys/ioctl.h HAVE_SYS_IOCTL_H)
check_include_file(sys/select.h HAVE_SYS_SELECT_H)
check_include_file(sys/stat.h HAVE_SYS_STAT_H)
check_include_file(sys/time.h HAVE_SYS_TIME_H)
check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(unistd.h HAVE_UNISTD_H)

check_struct_has_member("struct tm" tm_sec "sys/time.h" TM_IN_SYS_TIME LANGUAGE C)

pkg_check_modules(BSD REQUIRED IMPORTED_TARGET libbsd)
pkg_check_modules(NSL REQUIRED IMPORTED_TARGET libnsl)
find_package(Threads)

configure_file(config.h.in config.h)

# 0verkill target
add_executable(${PROJECT_NAME} client.c data.c sprite.c blit.c console.c hash.c time.c net.c error.c crc32.c md5.c md5hl.c kbd.c help.c)

target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_compile_definitions(${PROJECT_NAME} PRIVATE BASE_DIR="${BASE_DIR}")
target_link_libraries(${PROJECT_NAME} PkgConfig::BSD PkgConfig::NSL m)
if(TARGET Threads::Threads)
	target_link_libraries(${PROJECT_NAME} Threads::Threads)
	target_compile_definitions(${PROJECT_NAME} PRIVATE HAVE_LIBPTHREAD)
endif()
install(TARGETS ${PROJECT_NAME} RUNTIME COMPONENT runtime)

# x0verkill target
if(WITH_X)
	find_package(X11 REQUIRED)

	add_executable(x${PROJECT_NAME} client.c data.c blit.c sprite.c xinterface.c xkbd.c error.c md5.c md5hl.c hash.c time.c net.c crc32.c help.c)
	target_include_directories(x${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
	target_compile_definitions(x${PROJECT_NAME} PRIVATE BASE_DIR="${BASE_DIR}" XWINDOW)
	target_link_libraries(x${PROJECT_NAME} PkgConfig::BSD PkgConfig::NSL m X11::Xpm)
	if(TARGET Threads::Threads)
		target_link_libraries(x${PROJECT_NAME} Threads::Threads)
		target_compile_definitions(x${PROJECT_NAME} PRIVATE HAVE_LIBPTHREAD)
	endif()
	install(TARGETS x${PROJECT_NAME} RUNTIME COMPONENT runtime)
endif()

# 0verkill-avi target
add_executable(${PROJECT_NAME}-avi avi.c blit.c console.c kbd.c time.c avihelp.c error.c)
target_include_directories(${PROJECT_NAME}-avi PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
install(TARGETS ${PROJECT_NAME}-avi RUNTIME COMPONENT runtime)

# x0verkill-avi target
if(WITH_X)
	add_executable(x${PROJECT_NAME}-avi avi.c blit.c console.c kbd.c time.c avihelp.c error.c)
	target_include_directories(x${PROJECT_NAME}-avi PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
	target_compile_definitions(x${PROJECT_NAME}-avi PRIVATE XWINDOW)
	target_link_libraries(x${PROJECT_NAME}-avi X11::Xpm)
	install(TARGETS x${PROJECT_NAME}-avi RUNTIME COMPONENT runtime)
endif()

# 0verkill-bot target
add_executable(${PROJECT_NAME}-bot bot.c data.c hash.c time.c net.c error.c md5.c md5hl.c crc32.c sprite.c)
target_include_directories(${PROJECT_NAME}-bot PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_compile_definitions(${PROJECT_NAME}-bot PRIVATE BASE_DIR="${BASE_DIR}")
target_link_libraries(${PROJECT_NAME}-bot m)
install(TARGETS ${PROJECT_NAME}-bot RUNTIME COMPONENT runtime)

# 0verkill-server target
add_executable(${PROJECT_NAME}-server server.c data.c sprite.c blit.c console.c hash.c time.c kbd.c md5.c md5hl.c error.c net.c crc32.c)
target_include_directories(${PROJECT_NAME}-server PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_compile_definitions(${PROJECT_NAME}-server PRIVATE BASE_DIR="${BASE_DIR}")
target_link_libraries(${PROJECT_NAME}-server m)
install(TARGETS ${PROJECT_NAME}-server RUNTIME COMPONENT runtime)

# 0verkill-editor target
add_executable(${PROJECT_NAME}-editor editor.c data.c blit.c sprite.c console.c hash.c md5.c md5hl.c error.c time.c kbd.c)
target_include_directories(${PROJECT_NAME}-editor PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_compile_definitions(${PROJECT_NAME}-editor PRIVATE BASE_DIR="${BASE_DIR}")
install(TARGETS ${PROJECT_NAME}-editor RUNTIME COMPONENT runtime)

# x0verkill-editor target
if(WITH_X)
	add_executable(x${PROJECT_NAME}-editor editor.c data.c blit.c sprite.c console.c hash.c md5.c md5hl.c error.c time.c kbd.c)
	target_include_directories(x${PROJECT_NAME}-editor PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
	target_compile_definitions(x${PROJECT_NAME}-editor PRIVATE BASE_DIR="${BASE_DIR}" XWINDOW)
	target_link_libraries(x${PROJECT_NAME}-editor X11::Xpm)
	install(TARGETS x${PROJECT_NAME}-editor RUNTIME COMPONENT runtime)
endif()

# 0verkill-test_server target
add_executable(${PROJECT_NAME}-test_server test_server.c net.c data.c crc32.c blit.c sprite.c time.c hash.c md5.c md5hl.c error.c console.c kbd.c)
target_include_directories(${PROJECT_NAME}-test_server PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_compile_definitions(${PROJECT_NAME}-test_server PRIVATE BASE_DIR="${BASE_DIR}")
install(TARGETS ${PROJECT_NAME}-test_server RUNTIME COMPONENT runtime)

# Install data
install(DIRECTORY "data" DESTINATION "${BASE_DIR}")
install(DIRECTORY "grx" DESTINATION "${BASE_DIR}")
