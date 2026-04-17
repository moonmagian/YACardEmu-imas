cmake_minimum_required(VERSION 3.10)

project(YACardEmu)
cmake_policy(SET CMP0079 NEW)
cmake_policy(SET CMP0077 NEW)
set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)

add_executable(${PROJECT_NAME}
	Includes/main.cpp
	Includes/CardIo.cpp
	Includes/SerIo.cpp
	Includes/C1231LR.cpp
	Includes/C1231BR.cpp
	Includes/base64.cpp
	Includes/WebIo.cpp
	Includes/Printer.cpp

	Includes/utf8.h
	Includes/httplib.h
	Includes/CardIo.h
	Includes/SerIo.h
	Includes/C1231LR.h
	Includes/C1231BR.h
	Includes/base64.h
	Includes/WebIo.h
	Includes/Printer.h
)

find_package(PkgConfig REQUIRED)
find_package(Threads REQUIRED)
find_package(SPDLOG REQUIRED)
find_package(Iconv REQUIRED)
pkg_check_modules(SERIAL REQUIRED libserialport)
pkg_check_modules(SDL2 REQUIRED sdl2)
pkg_check_modules(SDL2_TTF REQUIRED SDL2_ttf)
pkg_check_modules(SDL2_IMAGE REQUIRED SDL2_image)

target_include_directories(
	${PROJECT_NAME}
	PUBLIC
	${SERIAL_INCLUDE_DIRS}
	${THREAD_INLCUDE_DIRS}
	${PROJECT_SOURCE_DIR}/Includes
	${PROJECT_SOURCE_DIR}/3rdparty/gulfs/include
	${SDL2_INCLUDE_DIRS}
	${IConv_INCLUDE_DIRS}
)

if (CMAKE_BUILD_TYPE STREQUAL "Release")
	include(CheckIPOSupported)
	check_ipo_supported(RESULT supported OUTPUT error)
	if (supported)
		set_property(TARGET ${PROJECT_NAME} PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
	endif()
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD 14)


target_compile_options(${PROJECT_NAME} PRIVATE -march=native -Wall -Wextra -Wpedantic)

target_link_libraries(${PROJECT_NAME}
	-lws2_32
	${CMAKE_THREAD_LIBS_INIT}
	spdlog::spdlog
	Iconv::Iconv
	${SERIAL_LIBRARIES}
	${SDL2_LIBRARIES}
	${SDL2_IMAGE_LIBRARIES}
	${SDL2_TTF_LIBRARIES}
)

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy_directory
		${CMAKE_CURRENT_SOURCE_DIR}/public
		${CMAKE_CURRENT_BINARY_DIR}/public
)

if (WIN32)
	add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy
			${CMAKE_CURRENT_SOURCE_DIR}/config.ini.sample
			${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_BUILD_TYPE}/config.ini.sample
	)
	add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy
			${CMAKE_CURRENT_SOURCE_DIR}/license.txt
			${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_BUILD_TYPE}/license.txt
	)
	add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
		COMMAND ${CMAKE_COMMAND} -E copy_directory
			${CMAKE_CURRENT_SOURCE_DIR}/public
			${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_BUILD_TYPE}/public
	)
endif()
