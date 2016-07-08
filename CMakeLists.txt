cmake_minimum_required(VERSION 3.2)

project(libplist LANGUAGES C CXX)

set(libplist_VERSION 0.1)

if(MSVC)
  set(CMAKE_DEBUG_POSTFIX "d")
endif()

find_package(libxml2 REQUIRED NO_MODULE)
find_package(iconv REQUIRED NO_MODULE)

add_library(libcnary STATIC 
  libcnary/include/iterator.h
  libcnary/include/list.h
  libcnary/include/node.h
  libcnary/include/node_iterator.h
  libcnary/include/node_list.h
  libcnary/include/object.h
  libcnary/cnary.c
  libcnary/iterator.c
  libcnary/list.c
  libcnary/node.c
  libcnary/node_iterator.c
  libcnary/node_list.c
)
target_include_directories(libcnary
  PRIVATE
  $<BUILD_INTERFACE:libcnary/;libcnary/include>
  PUBLIC
  $<INSTALL_INTERFACE:libcnary/include>
  INTERFACE
  $<INSTALL_INTERFACE:libcnary/include>
)

add_library(libplist
  include/plist/Array.h
  include/plist/Boolean.h
  include/plist/Data.h
  include/plist/Date.h
  include/plist/Dictionary.h
  include/plist/Integer.h
  include/plist/Key.h
  include/plist/Node.h
  include/plist/plist.h
  include/plist/plist++.h
  include/plist/Real.h
  include/plist/String.h
  include/plist/Structure.h
  include/plist/Uid.h
  src/Array.cpp
  src/base64.c
  src/base64.h
  src/Boolean.cpp
  src/bplist.c
  src/bytearray.c
  src/bytearray.h
  src/Data.cpp
  src/Date.cpp
  src/Dictionary.cpp
  src/hashtable.c
  src/hashtable.h
  src/Integer.cpp
  src/Key.cpp
  src/Node.cpp
  src/plist.c
  src/plist.h
  src/ptrarray.c
  src/ptrarray.h
  src/Real.cpp
  src/String.cpp
  src/Structure.cpp
  src/Uid.cpp
  src/xplist.c
)
target_include_directories(
  libplist PRIVATE
  $<BUILD_INTERFACE:include/;src/;libcnary/include/>
  INTERFACE
  $<INSTALL_INTERFACE:include/plist>
)

add_dependencies(libplist libcnary)
target_link_libraries(libplist PRIVATE libcnary libxml2::libxml2 iconv::iconv)
target_compile_definitions(libplist PRIVATE BUILDING_LIBPLIST)
if(NOT BUILD_SHARED_LIBS)
  target_compile_definitions(libplist PRIVATE PLIST_STATICLIB)
endif()

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/libplist-config-version.cmake
  VERSION ${libplist_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(TARGETS libplist EXPORT libplist
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib)


install(DIRECTORY
  include/plist
  DESTINATION include)

install(EXPORT libplist
  FILE
    libplist.cmake
  NAMESPACE
    libplist::
  DESTINATION
    lib/cmake/libplist
)
install(
  FILES
    cmake/libplist-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/libplist-config-version.cmake
  DESTINATION
    lib/cmake/libplist
)

if(MSVC AND BUILD_SHARED_LIBS)
  install(FILES
    $<TARGET_PDB_FILE:libplist>
    DESTINATION lib
  )
endif()