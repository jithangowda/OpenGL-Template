
cmake_minimum_required(VERSION 3.16)

set(CMAKE_CXX_STANDARD 17)


set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Release>:Release>")
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE) #link time optimization

# -------------------------------------------------------------------------------------
# compiler option
if(MSVC) 
	add_compile_options(/arch:AVX2) #make sure SIMD optimizations take place
endif()

if(CMAKE_COMPILER_IS_GNUCXX)
    # Set GCC specific flags for optimization
    add_compile_options(-march=native)  # Use native optimizations based on your system architecture
    add_compile_options(-flto)  # Link-time optimization
    add_compile_options(-O3)  # Enable maximum optimization
    add_compile_options(-Wall -Wextra)  # Enable more warnings
endif()
# -------------------------------------------------------------------------------------

project(OpenGLTemplate)

#glfw something bla bla
set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(GLFW_INSTALL OFF CACHE BOOL "" FORCE)
set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)

# Add Thirdparty LIbs
add_subdirectory(thirdparty/glfw-3.4)			#window oppener
add_subdirectory(thirdparty/glad)				#opengl loader
add_subdirectory(thirdparty/glm)				#math


#Add Sources
file(GLOB_RECURSE MY_SOURCES CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")

add_executable("${CMAKE_PROJECT_NAME}")

set_property(TARGET "${CMAKE_PROJECT_NAME}" PROPERTY CXX_STANDARD 17)

target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC GLFW_INCLUDE_NONE=1) 

# ----------------------------------------------------------------------------------
# Resource Paths
set(SHADER_VERT_PATH "${CMAKE_SOURCE_DIR}/resources/shader.vert")
set(SHADER_FRAG_PATH "${CMAKE_SOURCE_DIR}/resources/shader.frag")

target_compile_definitions(${PROJECT_NAME} PRIVATE
    SHADER_VERT_PATH="${SHADER_VERT_PATH}"
    SHADER_FRAG_PATH="${SHADER_FRAG_PATH}"
)
# ----------------------------------------------------------------------------------

target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC PRODUCTION_BUILD=0) 

target_sources("${CMAKE_PROJECT_NAME}" PRIVATE ${MY_SOURCES})

# ----------------------------------------------------------------------------------
if(MSVC) # If using the VS compiler...
	target_compile_definitions("${CMAKE_PROJECT_NAME}" PUBLIC _CRT_SECURE_NO_WARNINGS)
	#set_target_properties("${CMAKE_PROJECT_NAME}" PROPERTIES LINK_FLAGS "/SUBSYSTEM:WINDOWS /ENTRY:mainCRTStartup") #no console
	#set_property(TARGET "${CMAKE_PROJECT_NAME}" PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreadedDebug<$<CONFIG:Debug>:Debug>")
	#set_property(TARGET "${CMAKE_PROJECT_NAME}" PROPERTY MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Release>:Release>")

endif()

# If using GCC, no need for MSVC-specific settings
if(CMAKE_COMPILER_IS_GNUCXX)
    # Add any additional GCC flags if needed
    target_compile_options("${CMAKE_PROJECT_NAME}" PRIVATE -std=c++17)  # Ensure C++17 is used
endif()
# ----------------------------------------------------------------------------------

target_include_directories("${CMAKE_PROJECT_NAME}" PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include/")

target_link_libraries("${CMAKE_PROJECT_NAME}" PRIVATE 
	glm 
	glfw 
	glad
)
