cmake_minimum_required(VERSION 3.0)
SET (CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/")

include(LLVMTools)
llvm_find_config(3.4)
if (NOT LLVM_CONFIG_EXECUTABLE)
    message(FATAL_ERROR "typelib-clang-tlb-importer: llvm-config was not found, cannot continue.")
endif (NOT LLVM_CONFIG_EXECUTABLE)
llvm_get_config(LLVM_CXXFLAGS "--cxxflags")
string(REGEX REPLACE "-I[^ ]+" "" LLVM_CXXFLAGS ${LLVM_CXXFLAGS})
llvm_get_config(LLVM_LIBRARY_DIRS "--libdir")
llvm_get_config(LLVM_INCLUDE_DIRS "--includedir")

unset(LLVM_RAW_LIBRARIES CACHE)
list(APPEND LLVM_RAW_LIBRARIES
    clangEdit clangTooling clangDriver
    clangFrontend clangLex clangParse
    clangAnalysis clangSema clangRewriteCore
    clangSerialization clangEdit clangAST
    clangASTMatchers clangBasic clang)

llvm_get_config(llvm_config_libs "--libs"
    core option analysis support bitreader mcparser transformutils)
string(REPLACE "-l" "" llvm_config_libs "${llvm_config_libs}")
string(REPLACE " " ";" llvm_config_libs "${llvm_config_libs}")
list(APPEND LLVM_RAW_LIBRARIES ${llvm_config_libs})

foreach(llvm_lib ${LLVM_RAW_LIBRARIES})
    unset(llvm_lib_path CACHE)
    find_library(llvm_lib_path ${llvm_lib} PATHS ${LLVM_LIBRARY_DIRS})
    if (NOT llvm_lib_path)
        message(STATUS "${llvm_lib} not found in ${LLVM_LIBRARY_DIRS}")
        set(LLVM_LIBRARIES FALSE)
        break()
    else()
        list(APPEND LLVM_LIBRARIES ${llvm_lib_path})
    endif()
endforeach()

if (NOT LLVM_LIBRARIES)
    message(FATAL_ERROR "missing libs.")
endif()

add_definitions(${LLVM_CXXFLAGS})
include_directories(${LLVM_INCLUDE_DIRS})


add_executable(main
    main.cpp
)


target_link_libraries(main
    ${LLVM_LIBRARIES}
    pthread
    tinfo
    dl
    )
