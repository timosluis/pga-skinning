include(FetchContent)

# This tracks the latest commit on the master branch of the Klein
# repository. Instead of `origin/master`, you can specify a specific
# commit hash or tag.
FetchContent_Declare(
	klein
	GIT_REPOSITORY https://github.com/jeremyong/Klein.git
	GIT_TAG origin/master
)
FetchContent_MakeAvailable(klein)

# Function for building single example
function(buildExample EXAMPLE_NAME)
	SET(EXAMPLE_FOLDER ${CMAKE_CURRENT_SOURCE_DIR}/${EXAMPLE_NAME})
	message(STATUS "Generating project file for example in ${EXAMPLE_FOLDER}")
	# Main
	file(GLOB SOURCE *.cpp ${BASE_HEADERS} ${EXAMPLE_FOLDER}/*.cpp)
	SET(MAIN_CPP ${EXAMPLE_FOLDER}/${EXAMPLE_NAME}.cpp)
	if(EXISTS ${EXAMPLE_FOLDER}/main.cpp)
		SET(MAIN_CPP ${EXAMPLE_FOLDER}/main.cpp)
	ENDIF()
	if(EXISTS ${EXAMPLE_FOLDER}/${EXAMPLE_NAME}.h)
		SET(MAIN_HEADER ${EXAMPLE_FOLDER}/${EXAMPLE_NAME}.h)
	ENDIF()
	find_package(OpenMP)
	# imgui example requires additional source files
	IF(${EXAMPLE_NAME} STREQUAL "imgui")
		file(GLOB ADD_SOURCE "../external/imgui/*.cpp")
		SET(SOURCE ${SOURCE} ${ADD_SOURCE})
	ENDIF()
	# wayland requires additional source files
	IF(USE_WAYLAND_WSI)
		SET(SOURCE ${SOURCE} ${CMAKE_BINARY_DIR}/xdg-shell-client-protocol.h ${CMAKE_BINARY_DIR}/xdg-shell-protocol.c)
	ENDIF()
	# Add shaders
	set(SHADER_DIR_GLSL "../data/shaders/${EXAMPLE_NAME}")
	file(GLOB SHADERS_GLSL "${SHADER_DIR_GLSL}/*.vert" "${SHADER_DIR_GLSL}/*.frag" "${SHADER_DIR_GLSL}/*.comp" "${SHADER_DIR_GLSL}/*.geom" "${SHADER_DIR_GLSL}/*.tesc" "${SHADER_DIR_GLSL}/*.tese" "${SHADER_DIR_GLSL}/*.mesh" "${SHADER_DIR_GLSL}/*.task" "${SHADER_DIR_GLSL}/*.rgen" "${SHADER_DIR_GLSL}/*.rchit" "${SHADER_DIR_GLSL}/*.rmiss" "${SHADER_DIR_GLSL}/*.rcall")
	source_group("Shaders" FILES ${SHADERS_GLSL})
	# Add optional readme / tutorial
	file(GLOB README_FILES "${EXAMPLE_FOLDER}/*.md")
	if(WIN32)
		add_executable(${EXAMPLE_NAME} WIN32 ${MAIN_CPP} ${SOURCE} ${MAIN_HEADER} ${SHADERS_GLSL} ${README_FILES})
		target_link_libraries(${EXAMPLE_NAME} base ${Vulkan_LIBRARY} ${WINLIBS})
	else(WIN32)
		add_executable(${EXAMPLE_NAME} ${MAIN_CPP} ${SOURCE} ${MAIN_HEADER} ${SHADERS_GLSL} ${README_FILES})
		target_link_libraries(${EXAMPLE_NAME} base )
	endif(WIN32)

	set_target_properties(${EXAMPLE_NAME} PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
	if(OpenMP_CXX_FOUND)
		target_compile_options(${EXAMPLE_NAME} PRIVATE ${OpenMP_CXX_FLAGS})
		IF(${EXAMPLE_NAME} STREQUAL "texture3d")
			if(OpenMP_CXX_FOUND)
    			target_link_libraries(${EXAMPLE_NAME} OpenMP::OpenMP_CXX)
			endif()	
		endif()
	endif()

	# Now, you can use target_link_libraries(your_lib PUBLIC klein::klein)
	# If you can target SSE4.1 (~97% market penetration), you can link against
	# the target klein::klein_sse42 instead.
	target_link_libraries(${EXAMPLE_NAME} PUBLIC klein::klein)

	if(RESOURCE_INSTALL_DIR)
		install(TARGETS ${EXAMPLE_NAME} DESTINATION ${CMAKE_INSTALL_BINDIR})
	endif()
endfunction(buildExample)

buildExample(dqs)
buildExample(bis)
buildExample(lbs)
# buildExample(pgacayley)
