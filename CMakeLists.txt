cmake_minimum_required(VERSION 3.5)
project(glew VERSION 1.0.0 LANGUAGES C)
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR}/dist CACHE PATH "..." FORCE)
endif()

add_library(glew src/glew.c)
target_include_directories(glew
    PUBLIC
        $<INSTALL_INTERFACE:include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)

if( WIN32 )
	target_compile_definitions(glew PUBLIC
		-DWIN32_MEAN_AND_LEAN -DVC_EXTRALEAN -DGLEW_BUILD -D_CRT_SECURE_NO_WARNINGS 
		-DGLEW_NO_GLU 
	)
endif()

if( WIN32 )
	set( GL_LIBRARY opengl32 )
elseif( APPLE )
	find_library( GL_LIBRARY OpenGL )
	find_library( AGL_LIBRARY AGL )
	set( PLATFORM_SPECIFIC_LIBRARIES ${AGL_LIBRARY} )
else(  )
	set( GL_LIBRARY GL )
	set( PLATFORM_SPECIFIC_LIBRARIES X11 dl Xext )
endif(  )
target_link_libraries(glew ${GL_LIBRARY} ${PLATFORM_SPECIFIC_LIBRARIES} )

set(PROJ_CONFIG_DIR lib/cmake/glew)
set(PROJ_EXPORT_NAME GLEW)
set(PROJ_EXPORT_NAMESPACE "")
set(PROJ_TARGET glew)

install(TARGETS ${PROJ_TARGET}
    EXPORT  ${PROJ_EXPORT_NAME}Targets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)

install(DIRECTORY include/ DESTINATION include)

set_target_properties(${PROJ_TARGET} PROPERTIES EXPORT_NAME ${PROJ_EXPORT_NAME})

install(EXPORT ${PROJ_EXPORT_NAME}Targets
    FILE
        ${PROJ_EXPORT_NAME}Targets.cmake 
    NAMESPACE
        ${PROJ_EXPORT_NAMESPACE}
    DESTINATION
        ${PROJ_CONFIG_DIR}
)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJ_EXPORT_NAME}ConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(${PROJ_EXPORT_NAME}Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJ_EXPORT_NAME}Config.cmake
    INSTALL_DESTINATION ${PROJ_CONFIG_DIR}
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJ_EXPORT_NAME}Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJ_EXPORT_NAME}ConfigVersion.cmake
    DESTINATION ${PROJ_CONFIG_DIR}
)

export(EXPORT ${PROJ_EXPORT_NAME}Targets FILE ${CMAKE_CURRENT_BINARY_DIR}/${PROJ_EXPORT_NAME}Targets.cmake  NAMESPACE ${PROJ_EXPORT_NAME}::)


add_executable( glewinfo src/glewinfo.c)
target_compile_definitions(glewinfo PRIVATE -DGLEW_STATIC)
target_link_libraries( glewinfo glew)

add_executable( visualinfo src/visualinfo.c)
target_compile_definitions(visualinfo PRIVATE -DGLEW_STATIC)
target_link_libraries( visualinfo glew )