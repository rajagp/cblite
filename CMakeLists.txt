cmake_minimum_required (VERSION 3.1)
cmake_policy(VERSION 3.1)
project (LiteCoreTools)

if(ANDROID)
    return()
endif()

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_STANDARD 11)
set(LITECORE vendor/couchbase-lite-core/)

### CBLITE

aux_source_directory("cblite" CBLITE_SRC)
aux_source_directory("litecp" LITECP_SRC)
aux_source_directory("vendor/linenoise-ng/src" LINENOISE_SRC)
set(LOGCAT_SRC "cbl-log/cbl-log.cc")

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
endif()

add_subdirectory(${LITECORE})
get_directory_property(LITECORE_LIBRARIES_PRIVATE DIRECTORY ${LITECORE} DEFINITION LITECORE_LIBRARIES_PRIVATE)

add_executable(cblite
               ${CBLITE_SRC}
               ${LITECP_SRC}
               ${LINENOISE_SRC}
               Tool.cc
               ArgumentTokenizer.cc)

target_include_directories(cblite PRIVATE ${PROJECT_SOURCE_DIR}
                    litecp
                    vendor/linenoise-ng/include/
                    ${LITECORE}C/
                    ${LITECORE}C/include/
                    ${LITECORE}LiteCore/Support/
                    ${LITECORE}Replicator/			     # for CivetWebSocket.hh
                    ${LITECORE}vendor/fleece/API/
                    ${LITECORE}vendor/fleece/Fleece/Support/ # PlatformCompat.hh
                    )

if (APPLE)
    set(PLATFORM_LIBS z
		 			"-framework CoreFoundation" "-framework Security")
elseif(MSVC)
    set(PLATFORM_LIBS zlibstatic)
else()
    set(PLATFORM_LIBS pthread z dl
                	${ICU4C_COMMON} ${ICU4C_I18N} ${LIBCXX_LIB} ${LIBCXXABI_LIB})
endif()

target_link_libraries(cblite ${LITECORE_LIBRARIES_PRIVATE} LiteCoreREST_Static ${PLATFORM_LIBS})

### cbl-log

add_executable(cbl-log
               ${LOGCAT_SRC}
               ${LINENOISE_SRC}
               Tool.cc
               ArgumentTokenizer.cc)

target_include_directories(cbl-log PRIVATE
${PROJECT_SOURCE_DIR}
${LITECORE}C/include/
vendor/linenoise-ng/include/
${LITECORE}LiteCore/Support/
${LITECORE}vendor/fleece/API/
${LITECORE}vendor/fleece/Fleece/Support/ # PlatformCompat.hh
)
target_link_libraries(cbl-log ${LITECORE_LIBRARIES_PRIVATE} ${PLATFORM_LIBS})

### TESTS

include_directories(${LITECORE}vendor/fleece/vendor/catch)
include_directories(${PROJECT_SOURCE_DIR})
add_executable(cblitetest tests/TokenizerTest.cc
	 					  ArgumentTokenizer.cc)

include_directories(${LITECORE}LiteCore/Support/
    ${LITECORE}LiteCore/Storage/
    ${LITECORE}vendor/fleece/Fleece/Support/
    ${LITECORE}vendor/fleece/API/
    ${LITECORE}vendor/BLIP-Cpp/src/util
    ${LITECORE}C/include/)
add_executable(cbl-logtest tests/TokenizerTest.cc
    ArgumentTokenizer.cc
    ${LITECORE}LiteCore/tests/LogEncoderTest.cc)
target_link_libraries(cbl-logtest ${LITECORE_LIBRARIES_PRIVATE} ${PLATFORM_LIBS})
