cmake_minimum_required(VERSION 3.16.3)

set(LeapSDK_DIR D:/PROGRAMMING/CLion/leapMotion/LeapSDK/lib/cmake/LeapSDK)
message(STATUS ${LeapSDK_DIR} )

if (WIN32)
	set(ULTRALEAP_PATH_ROOT "$ENV{ProgramFiles}/Ultraleap")
elseif (UNIX)
	set(ULTRALEAP_PATH_ROOT "/usr/share/doc/ultraleap-hand-tracking-service/lib/cmake/LeapSDK")
endif()

find_package(LeapSDK
	5
	REQUIRED
	PATHS
		"${ULTRALEAP_PATH_ROOT}" )

if (UNIX)    
    find_package(Threads REQUIRED)    
endif (UNIX)

add_executable(
	leapc_example
	"leapc_main.c")

target_link_libraries(
	leapc_example
	PRIVATE
		LeapSDK::LeapC)

get_target_property(
	LEAPC_IMPORTED_CONFIG
	LeapSDK::LeapC
	IMPORTED_CONFIGURATIONS
)

get_target_property(
	LEAPC_SHARED_LIB_PATH
	LeapSDK::LeapC
	IMPORTED_LOCATION_${LEAPC_IMPORTED_CONFIG}
)

add_custom_command(
	TARGET
		leapc_example
	POST_BUILD
	COMMAND
		${CMAKE_COMMAND} -E copy
		${LEAPC_SHARED_LIB_PATH}
		$<TARGET_FILE_DIR:leapc_example>)

add_library(
	libExampleConnection
	OBJECT
	"ExampleConnection.c")


if (UNIX)
	target_link_libraries(
		libExampleConnection
		PRIVATE
		Threads::Threads)
endif()    

target_include_directories(
	libExampleConnection
	PUBLIC
	${CMAKE_CURRENT_SOURCE_DIR})

# Add targets for each sample file.
function(add_sample sample_name sample_source_file)

  add_executable(${sample_name} ${sample_source_file})
  set_property(TARGET ${sample_name} PROPERTY FOLDER "Samples")
  target_link_libraries(${sample_name} PUBLIC libExampleConnection)

endfunction()


add_sample("CallbackSample" "CallbackSample.c")
add_sample("ImageSample" "ImageSample.c")
add_sample("PollingSample" "PollingSample.c")
add_sample("InterpolationSample" "InterpolationSample.c")
if(NOT ANDROID)
	add_sample("MultiDeviceSample" "MultiDeviceSample.c")
endif()
