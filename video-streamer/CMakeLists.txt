project(video-streamer)

FILE(GLOB SOURCES *.cpp *.hpp)

add_executable(video-streamer ${SOURCES})

message(STATUS "SRT_INCLUDE_DIR: ${SRT_INCLUDE_DIR}")
message(STATUS "SRT_UTILS_INCLUDE_DIR: ${SRT_UTILS_INCLUDE_DIR}")
message(STATUS "CMAKE_BINARY_DIR: ${CMAKE_BINARY_DIR}")
message(STATUS "VIRTUAL_srtsupport: ${VIRTUAL_srtsupport}")

target_include_directories(video-streamer PUBLIC
	${SRT_INCLUDE_DIR}
	${SRT_UTILS_INCLUDE_DIR}
)

if (ENABLE_BONDING OR ENABLE_EXPERIMENTAL_BONDING)
	target_compile_options(video-streamer
		PRIVATE "-DENABLE_BONDING=1" "-DENABLE_EXPERIMENTAL_BONDING"
	)
endif()

if (ENABLE_CXX17)
	set(REQUIRE_CXX_VER 17)
	if (${CMAKE_CXX_COMPILER_ID} MATCHES "GNU|Intel")
		target_compile_options(video-streamer PRIVATE "-lstdc++fs")
		set(LINKSTDCPP_FS stdc++fs)
	endif()
	target_compile_options(video-streamer PRIVATE "-DENABLE_FILE_TRANSFER" "-DENABLE_CXX17")
	message(STATUS "C++17 features: ENABLED")
else()
	set(REQUIRE_CXX_VER 14)
	message(STATUS "C++17 features: DISABLED")
endif()

set (VIRTUAL_srtsupport $<TARGET_OBJECTS:srtsupport_virtual>)

target_link_libraries(video-streamer
	PRIVATE ${TARGET_srt}_static ${VIRTUAL_srtsupport} ${LINKSTDCPP_FS}
)

set_target_properties(video-streamer
	PROPERTIES
	CXX_STANDARD ${REQUIRE_CXX_VER}
	ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
	LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
	RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)