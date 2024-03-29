CMAKE_MINIMUM_REQUIRED(VERSION 3.6)

PROJECT("lua-gumbo")

ENABLE_TESTING()
FIND_PACKAGE(PythonInterp 2.7 REQUIRED)

# Build the Lua modules.
ADD_LIBRARY(TARGET_parse MODULE
            gumbo/parse.c
            lib/ascii.c
            lib/attribute.c
            lib/char_ref.c
            lib/error.c
            lib/foreign_attrs.c
            lib/parser.c
            lib/string_buffer.c
            lib/svg_attrs.c
            lib/svg_tags.c
            lib/tag.c
            lib/tag_lookup.c
            lib/tokenizer.c
            lib/utf8.c
            lib/util.c
            lib/vector.c)
TARGET_INCLUDE_DIRECTORIES(TARGET_parse
                           PRIVATE ${LUA_INCLUDE_DIR})
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	TARGET_LINK_LIBRARIES(TARGET_parse ${LUA_LIBRARIES})
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Set the name of the output file to "parse".
SET_TARGET_PROPERTIES(TARGET_parse PROPERTIES PREFIX "" OUTPUT_NAME "parse")

# On mingw link all compiler libraries static.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	SET_PROPERTY(TARGET TARGET_parse PROPERTY LINK_FLAGS "-static -static-libgcc -static-libstdc++")
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))


ADD_LIBRARY(TARGET_util MODULE
            gumbo/util.c
            lib/ascii.c)
TARGET_INCLUDE_DIRECTORIES(TARGET_util
                           PRIVATE ${LUA_INCLUDE_DIR})
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	TARGET_LINK_LIBRARIES(TARGET_util ${LUA_LIBRARIES})
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Set the name of the output file to "util".
SET_TARGET_PROPERTIES(TARGET_util PROPERTIES PREFIX "" OUTPUT_NAME "util")

# On mingw link all compiler libraries static.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	SET_PROPERTY(TARGET TARGET_util PROPERTY LINK_FLAGS "-static -static-libgcc -static-libstdc++")
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))


# Install the lua modules.
INSTALL(TARGETS TARGET_parse TARGET_util
        EXPORT EXPORT_package
        LIBRARY DESTINATION ${INSTALL_DIR_LUA_MODULES}/gumbo/
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)

# Add tests for the modules.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	# Here are the MinGW specific tests.
	ADD_TEST(NAME parse_MinGW_DLL_dependencies
	         COMMAND "${PYTHON_EXECUTABLE}" ${TEST_SCRIPT_FOLDER}/mingw_dll_dependencies.py -u lua5.1 -u lua5.2 -u lua5.3 -u lua5.4 $<TARGET_FILE:TARGET_parse>)
	ADD_TEST(NAME util_MinGW_DLL_dependencies
	         COMMAND "${PYTHON_EXECUTABLE}" ${TEST_SCRIPT_FOLDER}/mingw_dll_dependencies.py -u lua5.1 -u lua5.2 -u lua5.3 -u lua5.4 $<TARGET_FILE:TARGET_util>)
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Install all LUA scripts.
INSTALL(FILES gumbo.lua
        DESTINATION ${INSTALL_DIR_LUA_SCRIPTS})

INSTALL(FILES gumbo/Buffer.lua gumbo/constants.lua gumbo/sanitize.lua gumbo/Set.lua
        DESTINATION ${INSTALL_DIR_LUA_SCRIPTS}/gumbo)

INSTALL(FILES gumbo/dom/AttributeList.lua gumbo/dom/Attribute.lua gumbo/dom/ChildNode.lua gumbo/dom/Comment.lua gumbo/dom/DocumentFragment.lua gumbo/dom/Document.lua gumbo/dom/DocumentType.lua gumbo/dom/DOMTokenList.lua gumbo/dom/ElementList.lua gumbo/dom/Element.lua gumbo/dom/NodeList.lua gumbo/dom/Node.lua gumbo/dom/ParentNode.lua gumbo/dom/Text.lua gumbo/dom/util.lua
        DESTINATION ${INSTALL_DIR_LUA_SCRIPTS}/gumbo/dom)

INSTALL(FILES gumbo/serialize/html.lua gumbo/serialize/Indent.lua
        DESTINATION ${INSTALL_DIR_LUA_SCRIPTS}/gumbo/serialize)
