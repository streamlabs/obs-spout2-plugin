project(win-spout)

set(CMAKE_CXX_STANDARD 11)

if(MSVC)
	include_directories(deps/Spout2/SPOUTSDK/SpoutLibrary)
	include_directories(deps/Spout2/SPOUTSDK/SpoutDirectX/SpoutDX)
endif()

set(SPOUT_BUILD_CMT OFF CACHE BOOL "" FORCE)
set(SPOUT_BUILD_LIBRARY ON CACHE BOOL "" FORCE)
set(SPOUT_BUILD_SPOUTDX ON CACHE BOOL "" FORCE)
set(SKIP_INSTALL_ALL ON CACHE BOOL "" FORCE)
add_subdirectory(deps/Spout2 EXCLUDE_FROM_ALL)

if(MSVC)
	foreach(spout_target Spout Spout_static SpoutLibrary SpoutLibrary_static SpoutDX SpoutDX_static)
		if(TARGET ${spout_target})
			target_compile_options(${spout_target} PRIVATE /wd4828)
		endif()
	endforeach()
endif()

set(win-spout_HEADERS
	source/win-spout.h
)

set(win-spout_SOURCES
	source/win-spout.cpp
	source/win-spout-source.cpp
)

add_library(win-spout MODULE
	${win-spout_SOURCES}
	${win-spout_HEADERS})
add_library(OBS::spout ALIAS win-spout)

include_directories(
	"${CMAKE_SOURCE_DIR}/libobs"
)

target_link_libraries(win-spout PRIVATE
	OBS::libobs
	SpoutLibrary
	SpoutDX
	OBS::w32-pthreads
)
target_link_directories(win-spout PRIVATE
	$<TARGET_LINKER_FILE_DIR:SpoutLibrary>
	$<TARGET_LINKER_FILE_DIR:SpoutDX>
)
add_dependencies(win-spout Spout)

set(MODULE_DESCRIPTION "Spout2 plugin")

function(install_spout_dependencies)
	install(TARGETS Spout SpoutLibrary SpoutDX RUNTIME DESTINATION "${OBS_PLUGIN_DESTINATION}")
endfunction()

set_target_properties(win-spout PROPERTIES FOLDER "plugins")
set_target_properties_obs(win-spout PROPERTIES FOLDER plugins/win-spout PREFIX "")

install_spout_dependencies()
