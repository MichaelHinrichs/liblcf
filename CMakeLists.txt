cmake_minimum_required(VERSION 3.10...3.20 FATAL_ERROR)
if(${CMAKE_VERSION} VERSION_LESS 3.12)
	# Use most recent policies, even for older cmake
	cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
endif()

project(liblcf VERSION 0.6.2 LANGUAGES CXX)

# Deprecation handling. Remove these lines one release later.
SET(new_params "WITH_ICU;WITH_XML;UPDATE_MIMEDB;ENABLE_TOOLS;ENABLE_BENCHMARKS")
SET(old_params "DISABLE_ICU;DISABLE_XML;DISABLE_UPDATE_MIMEDB;DISABLE_TOOLS;DISABLE_BENCHMARKS")
foreach(x IN LISTS old_params)
	if(DEFINED ${x})
		list(FIND old_params ${x} index)
		list(GET new_params ${index} new_value)
		message(FATAL_ERROR "${x} was renamed to LIBLCF_${new_value} and the logic inverted.")
	endif()
endforeach()

# Compilation options
option(BUILD_SHARED_LIBS "Build shared library, disable for building the static library (default: ON)" ON)
option(LIBLCF_WITH_ICU "ICU encoding detection (when OFF fallback to iconv, not recommended, default: ON)" ON)
option(LIBLCF_WITH_XML "XML reading support (expat, default: ON)" ON)
option(LIBLCF_UPDATE_MIMEDB "Whether to run update-mime-database after install (default: ON)" ON)
option(LIBLCF_ENABLE_TOOLS "Whether to build the tools (default: ON)" ON)
option(LIBLCF_ENABLE_TESTS "Whetner to build the unit tests (default: ON)" ON)
option(LIBLCF_ENABLE_BENCHMARKS "Whether to build the benchmarks (default: OFF)" OFF)

set(CMAKE_DEBUG_POSTFIX "d" CACHE STRING "Override CMAKE_DEBUG_POSTFIX.")

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/builds/cmake/Modules)
include(ConfigureWindows)

# C++14 is required
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

# lcf library files
set(LCF_SOURCES
	src/dbarray.cpp
	src/encoder.cpp
	src/ini.cpp
	src/inireader.cpp
	src/ldb_equipment.cpp
	src/ldb_eventcommand.cpp
	src/ldb_parameters.cpp
	src/ldb_reader.cpp
	src/lmt_reader.cpp
	src/lmt_rect.cpp
	src/lmt_treemap.cpp
	src/lmu_movecommand.cpp
	src/lmu_reader.cpp
	src/lsd_reader.cpp
	src/reader_flags.cpp
	src/reader_lcf.cpp
	src/reader_struct.h
	src/reader_struct_impl.h
	src/reader_util.cpp
	src/reader_xml.cpp
	src/rpg_setup.cpp
	src/rpg_terms.cpp
	src/saveopt.cpp
	src/writer_lcf.cpp
	src/writer_xml.cpp
	src/generated/fwd_struct_impl.h
	src/generated/ldb_actor.cpp
	src/generated/ldb_animation.cpp
	src/generated/ldb_animationcelldata.cpp
	src/generated/ldb_animationframe.cpp
	src/generated/ldb_animationtiming.cpp
	src/generated/ldb_attribute.cpp
	src/generated/ldb_battlecommand.cpp
	src/generated/ldb_battlecommands.cpp
	src/generated/ldb_battleranimation.cpp
	src/generated/ldb_battleranimationitemskill.cpp
	src/generated/ldb_battleranimationpose.cpp
	src/generated/ldb_battleranimationweapon.cpp
	src/generated/ldb_chipset.cpp
	src/generated/ldb_class.cpp
	src/generated/ldb_commonevent.cpp
	src/generated/ldb_database.cpp
	src/generated/ldb_enemy.cpp
	src/generated/ldb_enemyaction.cpp
	src/generated/ldb_item.cpp
	src/generated/ldb_learning.cpp
	src/generated/ldb_music.cpp
	src/generated/ldb_skill.cpp
	src/generated/ldb_sound.cpp
	src/generated/ldb_state.cpp
	src/generated/ldb_switch.cpp
	src/generated/ldb_system.cpp
	src/generated/ldb_terms.cpp
	src/generated/ldb_terrain.cpp
	src/generated/ldb_terrain_flags.h
	src/generated/ldb_testbattler.cpp
	src/generated/ldb_troop.cpp
	src/generated/ldb_troopmember.cpp
	src/generated/ldb_trooppage.cpp
	src/generated/ldb_trooppagecondition.cpp
	src/generated/ldb_trooppagecondition_flags.h
	src/generated/ldb_variable.cpp
	src/generated/lmt_encounter.cpp
	src/generated/lmt_mapinfo.cpp
	src/generated/lmt_start.cpp
	src/generated/lmu_event.cpp
	src/generated/lmu_eventpage.cpp
	src/generated/lmu_eventpagecondition.cpp
	src/generated/lmu_eventpagecondition_flags.h
	src/generated/lmu_map.cpp
	src/generated/lmu_moveroute.cpp
	src/generated/lsd_save.cpp
	src/generated/lsd_saveactor.cpp
	src/generated/lsd_savecommonevent.cpp
	src/generated/lsd_saveeasyrpgdata.cpp
	src/generated/lsd_saveeventexecframe.cpp
	src/generated/lsd_saveeventexecstate.cpp
	src/generated/lsd_saveinventory.cpp
	src/generated/lsd_savemapevent.cpp
	src/generated/lsd_savemapeventbase.cpp
	src/generated/lsd_savemapinfo.cpp
	src/generated/lsd_savepanorama.cpp
	src/generated/lsd_savepartylocation.cpp
	src/generated/lsd_savepicture.cpp
	src/generated/lsd_savepicture_flags.h
	src/generated/lsd_savescreen.cpp
	src/generated/lsd_savesystem.cpp
	src/generated/lsd_savetarget.cpp
	src/generated/lsd_savetitle.cpp
	src/generated/lsd_savevehiclelocation.cpp
	src/generated/rpg_actor.cpp
	src/generated/rpg_animation.cpp
	src/generated/rpg_animationcelldata.cpp
	src/generated/rpg_animationframe.cpp
	src/generated/rpg_animationtiming.cpp
	src/generated/rpg_attribute.cpp
	src/generated/rpg_battlecommand.cpp
	src/generated/rpg_battlecommands.cpp
	src/generated/rpg_battleranimation.cpp
	src/generated/rpg_battleranimationitemskill.cpp
	src/generated/rpg_battleranimationpose.cpp
	src/generated/rpg_battleranimationweapon.cpp
	src/generated/rpg_chipset.cpp
	src/generated/rpg_class.cpp
	src/generated/rpg_commonevent.cpp
	src/generated/rpg_database.cpp
	src/generated/rpg_encounter.cpp
	src/generated/rpg_enemy.cpp
	src/generated/rpg_enemyaction.cpp
	src/generated/rpg_enums.cpp
	src/generated/rpg_equipment.cpp
	src/generated/rpg_event.cpp
	src/generated/rpg_eventcommand.cpp
	src/generated/rpg_eventpage.cpp
	src/generated/rpg_eventpagecondition.cpp
	src/generated/rpg_item.cpp
	src/generated/rpg_learning.cpp
	src/generated/rpg_map.cpp
	src/generated/rpg_mapinfo.cpp
	src/generated/rpg_movecommand.cpp
	src/generated/rpg_moveroute.cpp
	src/generated/rpg_music.cpp
	src/generated/rpg_parameters.cpp
	src/generated/rpg_rect.cpp
	src/generated/rpg_save.cpp
	src/generated/rpg_saveactor.cpp
	src/generated/rpg_savecommonevent.cpp
	src/generated/rpg_saveeasyrpgdata.cpp
	src/generated/rpg_saveeventexecframe.cpp
	src/generated/rpg_saveeventexecstate.cpp
	src/generated/rpg_saveinventory.cpp
	src/generated/rpg_savemapevent.cpp
	src/generated/rpg_savemapeventbase.cpp
	src/generated/rpg_savemapinfo.cpp
	src/generated/rpg_savepanorama.cpp
	src/generated/rpg_savepartylocation.cpp
	src/generated/rpg_savepicture.cpp
	src/generated/rpg_savescreen.cpp
	src/generated/rpg_savesystem.cpp
	src/generated/rpg_savetarget.cpp
	src/generated/rpg_savetitle.cpp
	src/generated/rpg_savevehiclelocation.cpp
	src/generated/rpg_skill.cpp
	src/generated/rpg_sound.cpp
	src/generated/rpg_start.cpp
	src/generated/rpg_state.cpp
	src/generated/rpg_switch.cpp
	src/generated/rpg_system.cpp
	src/generated/rpg_terms.cpp
	src/generated/rpg_terrain.cpp
	src/generated/rpg_testbattler.cpp
	src/generated/rpg_treemap.cpp
	src/generated/rpg_troop.cpp
	src/generated/rpg_troopmember.cpp
	src/generated/rpg_trooppage.cpp
	src/generated/rpg_trooppagecondition.cpp
	src/generated/rpg_variable.cpp
)

set(LCF_HEADERS
	src/lcf/context.h
	src/lcf/dbarray.h
	src/lcf/dbarrayalloc.h
	src/lcf/dbbitarray.h
	src/lcf/dbstring.h
	src/lcf/encoder.h
	src/lcf/enum_tags.h
	src/lcf/flag_set.h
	src/lcf/ini.h
	src/lcf/inireader.h
	src/lcf/ldb/reader.h
	src/lcf/lmt/reader.h
	src/lcf/lmu/reader.h
	src/lcf/lsd/reader.h
	src/lcf/reader_lcf.h
	src/lcf/reader_util.h
	src/lcf/reader_xml.h
	src/lcf/saveopt.h
	src/lcf/scope_guard.h
	src/lcf/span.h
	src/lcf/string_view.h
	src/lcf/writer_lcf.h
	src/lcf/writer_xml.h
	src/generated/lcf/ldb/chunks.h
	src/generated/lcf/lmt/chunks.h
	src/generated/lcf/lmu/chunks.h
	src/generated/lcf/lsd/chunks.h
	src/generated/lcf/rpg/actor.h
	src/generated/lcf/rpg/animation.h
	src/generated/lcf/rpg/animationcelldata.h
	src/generated/lcf/rpg/animationframe.h
	src/generated/lcf/rpg/animationtiming.h
	src/generated/lcf/rpg/attribute.h
	src/generated/lcf/rpg/battlecommand.h
	src/generated/lcf/rpg/battlecommands.h
	src/generated/lcf/rpg/battleranimation.h
	src/generated/lcf/rpg/battleranimationitemskill.h
	src/generated/lcf/rpg/battleranimationpose.h
	src/generated/lcf/rpg/battleranimationweapon.h
	src/generated/lcf/rpg/chipset.h
	src/generated/lcf/rpg/class.h
	src/generated/lcf/rpg/commonevent.h
	src/generated/lcf/rpg/database.h
	src/generated/lcf/rpg/encounter.h
	src/generated/lcf/rpg/enemy.h
	src/generated/lcf/rpg/enemyaction.h
	src/generated/lcf/rpg/equipment.h
	src/generated/lcf/rpg/event.h
	src/generated/lcf/rpg/eventcommand.h
	src/generated/lcf/rpg/eventpage.h
	src/generated/lcf/rpg/eventpagecondition.h
	src/generated/lcf/rpg/fwd.h
	src/generated/lcf/rpg/item.h
	src/generated/lcf/rpg/learning.h
	src/generated/lcf/rpg/map.h
	src/generated/lcf/rpg/mapinfo.h
	src/generated/lcf/rpg/movecommand.h
	src/generated/lcf/rpg/moveroute.h
	src/generated/lcf/rpg/music.h
	src/generated/lcf/rpg/parameters.h
	src/generated/lcf/rpg/rect.h
	src/generated/lcf/rpg/save.h
	src/generated/lcf/rpg/saveactor.h
	src/generated/lcf/rpg/savecommonevent.h
	src/generated/lcf/rpg/saveeasyrpgdata.h
	src/generated/lcf/rpg/saveeventexecframe.h
	src/generated/lcf/rpg/saveeventexecstate.h
	src/generated/lcf/rpg/saveinventory.h
	src/generated/lcf/rpg/savemapevent.h
	src/generated/lcf/rpg/savemapeventbase.h
	src/generated/lcf/rpg/savemapinfo.h
	src/generated/lcf/rpg/savepanorama.h
	src/generated/lcf/rpg/savepartylocation.h
	src/generated/lcf/rpg/savepicture.h
	src/generated/lcf/rpg/savescreen.h
	src/generated/lcf/rpg/savesystem.h
	src/generated/lcf/rpg/savetarget.h
	src/generated/lcf/rpg/savetitle.h
	src/generated/lcf/rpg/savevehiclelocation.h
	src/generated/lcf/rpg/skill.h
	src/generated/lcf/rpg/sound.h
	src/generated/lcf/rpg/start.h
	src/generated/lcf/rpg/state.h
	src/generated/lcf/rpg/switch.h
	src/generated/lcf/rpg/system.h
	src/generated/lcf/rpg/terms.h
	src/generated/lcf/rpg/terrain.h
	src/generated/lcf/rpg/testbattler.h
	src/generated/lcf/rpg/treemap.h
	src/generated/lcf/rpg/troop.h
	src/generated/lcf/rpg/troopmember.h
	src/generated/lcf/rpg/trooppage.h
	src/generated/lcf/rpg/trooppagecondition.h
	src/generated/lcf/rpg/variable.h
	src/lcf/third_party/span.h
	src/lcf/third_party/string_view.h
)

add_library(lcf ${LCF_SOURCES} ${LCF_HEADERS})

# IDE source grouping
foreach(SG LDB LMT LMU LSD RPG THIRD_PARTY)
	string(TOLOWER ${SG} LSG)
	source_group("Source Files\\generated\\${SG}" REGULAR_EXPRESSION "generated/${LSG}_.*\\.cpp")
	source_group("Header Files\\generated\\${SG}" REGULAR_EXPRESSION "generated/lcf/${LSG}/.*\\.h")
	source_group("Source Files\\${SG}" REGULAR_EXPRESSION "${LSG}_.*\\.cpp")
	source_group("Header Files\\${SG}" REGULAR_EXPRESSION "/${LSG}/.*\\.h")
endforeach()

# includes
target_include_directories(
	lcf PUBLIC
	${CMAKE_CURRENT_BINARY_DIR}/src
	${CMAKE_CURRENT_SOURCE_DIR}/src
	${CMAKE_CURRENT_SOURCE_DIR}/src/generated
)

# Optimize floating point math functions into intrinsics and don't check or set errno (i.e. sqrt(-1))
if (NOT CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
	target_compile_options(lcf PUBLIC "-fno-math-errno")
endif()

# endianess checking
if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
	include(TestBigEndian)
	test_big_endian(WORDS_BIGENDIAN)
	if(WORDS_BIGENDIAN)
		target_compile_definitions(lcf PRIVATE WORDS_BIGENDIAN=1)
	endif()
endif()

# Add optional library name postfix
set_property(TARGET lcf PROPERTY DEBUG_POSTFIX "${CMAKE_DEBUG_POSTFIX}")

# Export symbols for DLL
set_property(TARGET lcf PROPERTY WINDOWS_EXPORT_ALL_SYMBOLS ON)

# icu or fallback to iconv
if(LIBLCF_WITH_ICU)
	find_package(ICU COMPONENTS i18n uc data REQUIRED)
	target_link_libraries(lcf ICU::i18n ICU::uc ICU::data)
	list(APPEND LIBLCF_DEPS "icu-i18n")
	set(LCF_SUPPORT_ICU "1")
else()
	find_package(Iconv REQUIRED)
	target_link_libraries(lcf Iconv::Iconv)
endif()

# expat
if(LIBLCF_WITH_XML)
	find_package(expat CONFIG)
	if(expat_FOUND)
		target_link_libraries(lcf expat::expat)
	else()
		# Fallback to old expat detection
		find_package(EXPAT REQUIRED)
		target_link_libraries(lcf EXPAT::EXPAT)
	endif()
	list(APPEND LIBLCF_DEPS "expat")
	set(LCF_SUPPORT_XML "1")
endif()

# mime types
if(LIBLCF_UPDATE_MIMEDB AND NOT CMAKE_CROSSCOMPILING)
	find_program(UPDATE_MIME_DATABASE update-mime-database)
endif()

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/builds/cmake.config.h.in" "src/lcf/config.h" @ONLY)
target_sources(lcf PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/src/lcf/config.h)

# .so version
set_property(TARGET lcf PROPERTY SOVERSION 0)

# installation
include(GNUInstallDirs)

# pkg-config file generation
set(LCF_LIBDIR ${CMAKE_INSTALL_LIBDIR})
if(IS_ABSOLUTE ${LCF_LIBDIR})
	file(RELATIVE_PATH LCF_LIBDIR ${CMAKE_INSTALL_PREFIX} ${LCF_LIBDIR})
endif()
set(LCF_INCLUDEDIR ${CMAKE_INSTALL_INCLUDEDIR})
if(IS_ABSOLUTE ${LCF_INCLUDEDIR})
	file(RELATIVE_PATH LCF_INCLUDEDIR ${CMAKE_INSTALL_PREFIX} ${LCF_INCLUDEDIR})
endif()
set(PACKAGE_TARNAME ${PROJECT_NAME})
set(PACKAGE_VERSION ${PROJECT_VERSION})
set(prefix "${CMAKE_INSTALL_PREFIX}")
set(exec_prefix "\${prefix}")
set(libdir "\${exec_prefix}/${LCF_LIBDIR}")
set(includedir "\${prefix}/${LCF_INCLUDEDIR}")
string(REPLACE ";" " " AX_PACKAGE_REQUIRES_PRIVATE "${LIBLCF_DEPS}")
configure_file(builds/${PROJECT_NAME}.pc.in builds/${PROJECT_NAME}.pc @ONLY)

# Cmake-config file generation
configure_file(builds/${PROJECT_NAME}-config.cmake.in builds/${PROJECT_NAME}-config.cmake @ONLY)

install(
	TARGETS lcf
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

set(LCF_INSTALL_HEADERS ${LCF_HEADERS} ${CMAKE_CURRENT_BINARY_DIR}/src/lcf/config.h)
foreach(S ldb lmt lmu lsd rpg third_party)
	set(SUBDIR_HEADERS ${LCF_INSTALL_HEADERS})
	list(FILTER SUBDIR_HEADERS INCLUDE REGEX "/${S}/")
	install(FILES ${SUBDIR_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/lcf/${S})
	list(REMOVE_ITEM LCF_INSTALL_HEADERS ${SUBDIR_HEADERS})
endforeach()
install(FILES ${LCF_INSTALL_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/lcf)

install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/builds/${PROJECT_NAME}.pc
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)

install(
	FILES
	  ${CMAKE_CURRENT_BINARY_DIR}/builds/liblcf-config.cmake
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/liblcf)

# mime types
install(
	FILES ${CMAKE_CURRENT_SOURCE_DIR}/mime/${PROJECT_NAME}.xml
	DESTINATION ${CMAKE_INSTALL_DATADIR}/mime/packages
)
if(LIBLCF_UPDATE_MIMEDB AND UPDATE_MIME_DATABASE)
	install(CODE
		"execute_process(COMMAND ${UPDATE_MIME_DATABASE}
			\$ENV{DESTDIR}\${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_DATADIR}/mime)"
	)
endif()

# tests
if(LIBLCF_ENABLE_TESTS)
	file(GLOB TEST_FILES ${CMAKE_CURRENT_SOURCE_DIR}/tests/*.cpp)
	add_executable(test_runner_lcf EXCLUDE_FROM_ALL ${TEST_FILES})
	set_target_properties(test_runner_lcf PROPERTIES OUTPUT_NAME "test_runner")
	target_link_libraries(test_runner_lcf lcf)
	add_custom_target(check_lcf COMMAND test_runner_lcf)
	if(NOT TARGET check)
		add_custom_target(check)
	endif()
	add_dependencies(check_lcf test_runner_lcf)
	add_dependencies(check check_lcf)
endif()

# benchmarks
if(LIBLCF_ENABLE_BENCHMARKS)
	file(GLOB BENCH_FILES ${CMAKE_CURRENT_SOURCE_DIR}/bench/*.cpp)
	foreach(i ${BENCH_FILES})
		get_filename_component(name "${i}" NAME_WE)
		add_executable(bench_${name} ${i})
		target_link_libraries(bench_${name} lcf)
	endforeach()
endif()

# tools
if(LIBLCF_ENABLE_TOOLS)
	add_custom_target(tools)
	add_dependencies(tools lcf)
	list(APPEND TOOLS lcfstrings)
	if(LIBLCF_WITH_XML)
		list(APPEND TOOLS lcf2xml)
	endif()
	foreach(tool ${TOOLS})
		add_executable(${tool} tools/${tool}.cpp)
		target_link_libraries(${tool} lcf)
		add_dependencies(tools ${tool})
		install(TARGETS ${tool} RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
	endforeach()
endif()

# Doxygen documentation
find_package(Doxygen)
if(DOXYGEN_FOUND)
	add_custom_target(liblcf_doc
		${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/builds/Doxyfile
		DEPENDS lcf
		WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/builds
		COMMENT "Generating API documentation with Doxygen" VERBATIM)
else()
	message(STATUS "This means no API documentation is generated.")
endif()
