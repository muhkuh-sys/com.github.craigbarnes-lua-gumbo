cmake_minimum_required(VERSION 3.7)

PROJECT("lua-gumbo")

INCLUDE(ExternalProject)

SET(VERSION_EXTERNAL "0.5")
SET(VERSION_DEV      "")
SET(VERSION_PACKAGE  "1")
SET(PROJECT_VERSION  "${VERSION_EXTERNAL}.${VERSION_PACKAGE}")

# Get the VCS version for the jonchki configuration.
INCLUDE(${CMAKE_HOME_DIRECTORY}/cmake/version.cmake)

# Filter the jonchki configuration.
CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/lua-gumbo.xml
               ${CMAKE_BINARY_DIR}/lua${BUILDCFG_LUA_VERSION}-lua-gumbo-${PROJECT_VERSION}.xml
               @ONLY)

#----------------------------------------------------------------------------
#
# Build the project.
#

IF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))
	# Select the LUA version.
	INCLUDE(${CMAKE_HOME_DIRECTORY}/cmake/muhkuh_lua.cmake)

	MESSAGE("INSTALL_DIR_LUA_MODULES=${INSTALL_DIR_LUA_MODULES}")
	SET(PRJ_CMAKE_ARGS "")
	LIST(APPEND PRJ_CMAKE_ARGS "-DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")
	LIST(APPEND PRJ_CMAKE_ARGS "-DCMAKE_INSTALL_PREFIX=${CMAKE_CURRENT_BINARY_DIR}/install")
	LIST(APPEND PRJ_CMAKE_ARGS "-DINSTALL_DIR_LUA_MODULES=${INSTALL_DIR_LUA_MODULES}")
	LIST(APPEND PRJ_CMAKE_ARGS "-DINSTALL_DIR_LUA_SCRIPTS=${INSTALL_DIR_LUA_SCRIPTS}")
	LIST(APPEND PRJ_CMAKE_ARGS "-DLUA_LIBRARIES=${LUA_LIBRARIES}")
	LIST(APPEND PRJ_CMAKE_ARGS "-DLUA_INCLUDE_DIR=${LUA_INCLUDE_DIR}")

	ExternalProject_Add(TARGET_lua_gumbo
	                    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/lua-gumbo
	                    URL ${CMAKE_CURRENT_SOURCE_DIR}/lua-gumbo-${VERSION_EXTERNAL}${VERSION_DEV}.tar.gz
	                    URL_HASH SHA1=24d582efb098fab4726e0ed15f27745879cd2afa
	                    PATCH_COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/copy_over ${CMAKE_CURRENT_BINARY_DIR}/lua-gumbo/src/TARGET_lua_gumbo
	                    CMAKE_ARGS ${PRJ_CMAKE_ARGS}
	                    TEST_BEFORE_INSTALL 1
	                    TEST_COMMAND ${CMAKE_MAKE_PROGRAM} test
	                    INSTALL_COMMAND ${CMAKE_MAKE_PROGRAM} install
	)

	#----------------------------------------------------------------------------
	#
	# Build a package for the selected platform.
	#

	SET(CPACK_OUTPUT_FILE_PREFIX ${CMAKE_BINARY_DIR}/../../repository/com/github/craigbarnes/lua-gumbo/${PROJECT_VERSION})
	SET(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
	SET(CPACK_PACKAGE_FILE_NAME "lua${BUILDCFG_LUA_VERSION}-lua-gumbo-${PROJECT_VERSION}-${JONCHKI_PLATFORM}")

	SET(CPACK_GENERATOR "TXZ")
	INCLUDE(CPack)

	IF(${JONCHKI_PLATFORM_DIST_ID} STREQUAL "windows")
		# Package a windows build.
		CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/install_windows.lua
		               ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
		               @ONLY)

		INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
		        DESTINATION .)
	ELSE(${JONCHKI_PLATFORM_DIST_ID} STREQUAL "windows")
		# Package a linux build.
		CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/install_linux.lua
		               ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
		               @ONLY)

		INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
		        DESTINATION .)
	ENDIF(${JONCHKI_PLATFORM_DIST_ID} STREQUAL "windows")

	INSTALL(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/install/lua
	        DESTINATION .)
	INSTALL(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/install/lua_plugins
	        DESTINATION .)

	CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/lua-gumbo.xml
	               ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-gumbo-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
	               @ONLY)


	#-----------------------------------------------------------------------------
	#
	# Create a custom target named "pack".
	# It builds the package and creates the hashes.
	#
	ADD_CUSTOM_TARGET(pack
	                  COMMAND ${CMAKE_MAKE_PROGRAM}
	                  COMMAND ${CMAKE_MAKE_PROGRAM} package
	                  COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-gumbo-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
	                  COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-gumbo-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.tar.xz
	                  COMMENT "Pack the artifact and generate the hashes...")


	#-----------------------------------------------------------------------------
	#
	# Filter the pom.xml file.
	#
	SET(POM_ARTIFACT_ID  "lua-gumbo")
	SET(POM_GROUP_ID     "com.github.craigbarnes")
	SET(POM_VERSION      "${PROJECT_VERSION}")
	SET(POM_PACKAGING    "tar.xz")
	CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/pom.xml
	               ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-lua-gumbo-${PROJECT_VERSION}.pom
	               @ONLY)
ENDIF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))
