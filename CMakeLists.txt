cmake_minimum_required(VERSION 3.25)

set (DEV_BIN "$ENV{DEV_ROOT}/bin")

# Amalgamation file name
set (SQLITENAME sqlite-amalgamation-3430200)

project (sqlite3)

# Dwonload and extract SQLite source files
set (SQLITE_AMALGAMATION_URI "https://www.sqlite.org/2023/${SQLITENAME}.zip")
set (SQLITE_AMALGAMATION_FILE ${CMAKE_SOURCE_DIR}/build/${SQLITENAME}.zip)
if (NOT EXISTS ${SQLITE_AMALGAMATION_FILE})
  message ("Downloading ${SQLITE_AMALGAMATION_FILE} from ${SQLITE_AMALGAMATION_URI}")
  file (DOWNLOAD ${SQLITE_AMALGAMATION_URI} ${SQLITE_AMALGAMATION_FILE})
  file (ARCHIVE_EXTRACT INPUT ${SQLITE_AMALGAMATION_FILE})
  configure_file (${CMAKE_CURRENT_BINARY_DIR}/${SQLITENAME}/sqlite3.h ${CMAKE_SOURCE_DIR}/include/sqlite3/sqlite3.h COPYONLY)
  configure_file (${CMAKE_CURRENT_BINARY_DIR}/${SQLITENAME}/sqlite3ext.h ${CMAKE_SOURCE_DIR}/include/sqlite3/sqlite3ext.h COPYONLY)
  configure_file (${CMAKE_CURRENT_BINARY_DIR}/${SQLITENAME}/sqlite3.c ${CMAKE_SOURCE_DIR}/src/sqlite3.c COPYONLY)
  configure_file (${CMAKE_CURRENT_BINARY_DIR}/${SQLITENAME}/shell.c ${CMAKE_SOURCE_DIR}/src/shell.c COPYONLY)
endif ()

include_directories (include/sqlite3)

# Build SQLITE.DLL library
add_library (sqlite SHARED src/sqlite3.c)

# Build SQLITE3.LIB static libary
add_library (sqlite3 STATIC src/sqlite3.c)
set_target_properties (sqlite3 PROPERTIES
  ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib/${CMAKE_C_COMPILER_ARCHITECTURE_ID}/${CMAKE_BUILD_TYPE}
  COMPILE_DEFINITIONS "SQLITE_OMIT_UTF16;SQLITE_ENABLE_COLUMN_METADATA"
)

# Build SQLITE3 shell executable
add_executable (shell src/shell.c src/sqlite3.c)
set_target_properties (shell PROPERTIES
  OUTPUT_NAME sqlite3
)

# Copy 64-bit SQLite shell and DLL to ${DEV_ROOT}/bin folder
if (CMAKE_VS_PLATFORM_NAME MATCHES "x64" )
  set(RELEASE_CMD "${CMAKE_COMMAND}")
  add_custom_command(
    TARGET shell POST_BUILD
    COMMAND $<$<CONFIG:Release>:${RELEASE_CMD}> ARGS -E copy $<TARGET_FILE:shell> ${DEV_BIN}/$<TARGET_FILE_NAME:shell>
    VERBATIM
  )
  add_custom_command(
    TARGET sqlite POST_BUILD
    COMMAND $<$<CONFIG:Release>:${RELEASE_CMD}> ARGS -E copy $<TARGET_FILE:sqlite> ${DEV_BIN}/$<TARGET_FILE_NAME:sqlite>
    VERBATIM
  )
endif ()
