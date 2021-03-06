cmake_minimum_required(VERSION 2.6)

# set project name from current directory
get_filename_component(BASENAME ${CMAKE_SOURCE_DIR} NAME)
project(${BASENAME})

set(CMAKE_BUILD_TYPE DEBUG)
set(CMAKE_C_FLAGS "-Wall")
set(CMAKE_CXX_FLAGS "-Wall")

add_custom_target(git_update
    COMMAND git submodule init
    COMMAND git submodule update
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
    )

# define executable to build
include_directories("${PROJECT_BINARY_DIR}/inc")
aux_source_directory(src SRC_FILES)
add_executable(${PROJECT_NAME} ${SRC_FILES})
add_dependencies(${PROJECT_NAME} git_update)

# v4l2wrapper
include_directories("${PROJECT_BINARY_DIR}/v4l2wrapper/inc")
set_source_files_properties(${PROJECT_BINARY_DIR}/v4l2wrapper/src/V4l2Capture.cpp PROPERTIES GENERATED 1)
set_source_files_properties(${PROJECT_BINARY_DIR}/v4l2wrapper/src/V4l2MmapCapture.cpp PROPERTIES GENERATED 1)
set_source_files_properties(${PROJECT_BINARY_DIR}/v4l2wrapper/src/V4l2ReadCapture.cpp PROPERTIES GENERATED 1)

add_library(v4l2wrapper
    STATIC
    v4l2wrapper/src/V4l2Capture.cpp
    v4l2wrapper/src/V4l2MmapCapture.cpp
    v4l2wrapper/src/V4l2ReadCapture.cpp
    )
target_link_libraries(${PROJECT_NAME} v4l2wrapper)
add_dependencies(v4l2wrapper git_update)

#pthread
find_package (Threads)
target_link_libraries (${PROJECT_NAME} ${CMAKE_THREAD_LIBS_INIT}) 

# LOG4CPP
find_path(LOG4CPP_INCLUDE_DIR log4cpp/Category.hh)
if (NOT LOG4CPP_INCLUDE_DIR)
        message(STATUS "Could not find the liblog4cpp5-dev.")
        EXEC_PROGRAM("sudo apt-get install liblog4cpp5-dev")
        find_path(V4L2_INCLUDE_DIR log4cpp/Category.hh)
endif (NOT LOG4CPP_INCLUDE_DIR)
include_directories(${LOG4CPP_INCLUDE_DIR}) 

find_library(LOG4CPP_LIBRARY log4cpp)
target_link_libraries(${PROJECT_NAME} "${LOG4CPP_LIBRARY}")

# V4L2
find_path(V4L2_INCLUDE_DIR libv4l2.h)
if (NOT V4L2_INCLUDE_DIR)
	message(STATUS "Could not find the libv4l-dev.")
	EXEC_PROGRAM("sudo apt-get install libv4l-dev")
	find_path(V4L2_INCLUDE_DIR libv4l2.h)
endif (NOT V4L2_INCLUDE_DIR)
include_directories(${V4L2_INCLUDE_DIR}) 

find_library(V4L2_LIBRARY v4l2)
target_link_libraries(${PROJECT_NAME} "${V4L2_LIBRARY}")

# live555
find_path(LIBLIVE555_INCLUDE_DIR liveMedia/liveMedia.hh)
if (NOT LIBLIVE555_INCLUDE_DIR)
        message(STATUS "Could not find the liblivemedia-dev")
        EXEC_PROGRAM("sudo apt-get install liblivemedia-dev")
        find_path(LIBLIVE555_INCLUDE_DIR liveMedia/liveMedia.hh)
endif (NOT LIBLIVE555_INCLUDE_DIR)
foreach (LIBLIVE555_module groupsock liveMedia UsageEnvironment BasicUsageEnvironment)
	include_directories(${LIBLIVE555_INCLUDE_DIR}/${LIBLIVE555_module}) 
endforeach (LIBLIVE555_module)

set(LIBLIVE555_LIBRARIES "")
foreach (LIBLIVE555_module groupsock liveMedia UsageEnvironment BasicUsageEnvironment)
	find_library( ${LIBLIVE555_module}_LIBRARY ${LIBLIVE555_module} )
	if ( ${LIBLIVE555_module}_LIBRARY )
		set(LIBLIVE555_LIBRARIES ${LIBLIVE555_LIBRARIES} ${${LIBLIVE555_module}_LIBRARY})
	endif ( ${LIBLIVE555_module}_LIBRARY )
endforeach (LIBLIVE555_module)
target_link_libraries(${PROJECT_NAME} ${LIBLIVE555_LIBRARIES})

# package
install (TARGETS ${PROJECT_NAME} DESTINATION bin)

SET(CPACK_GENERATOR "DEB")
SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "Michel Promonet") 
SET(CPACK_PACKAGE_CONTACT "michel.promonet@free.fr") 

INCLUDE(CPack)

