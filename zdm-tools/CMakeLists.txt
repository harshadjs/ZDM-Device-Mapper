#* ---------------------------------------------------------------------------------------------
#* Copyright (c) 2015 Seagate Technologies
#* All rights reserved.
#* ---------------------------------------------------------------------------------------------

cmake_minimum_required (VERSION 2.8.12)
project (zdm_tools)

if (EXISTS /usr/include/linux/blkzoned.h)
   message ("System provided blkzoned.h GOOD.")
   add_definitions(-DHAVE_BLKZONED_H)
else (EXISTS /usr/include/linux/blkzoned.h)
   message ("No blkzonedi.h using local verson.")
   include_directories ("${PROJECT_SOURCE_DIR}/uapi-missing")
   add_definitions(-DHAVE_BLKZONED_H)
endif (EXISTS /usr/include/linux/blkzoned.h)

if (EXISTS /usr/lib/libsgutils2.a)
   message ("sg3 utils found..")
elseif (EXISTS /usr/lib/x86_64-linux-gnu/libsgutils2.a)
   message ("sg3 x64 utils found..")
else (EXISTS /usr/lib/libsgutils2.a)
   message ("Need libsg3utils-dev package.")
endif (EXISTS /usr/lib/libsgutils2.a)

INCLUDE(FindPkgConfig)

pkg_check_modules(UUID REQUIRED uuid)
pkg_check_modules(LIBSSL REQUIRED libssl)
pkg_check_modules(LIBUDEV REQUIRED libudev)

include_directories ("${PROJECT_SOURCE_DIR}/h")
include_directories ("${PROJECT_BINARY_DIR}/h")

# For O_DIRECT
add_definitions(-D_GNU_SOURCE)

# For pread64/pwrite64
add_definitions(-D_LARGEFILE64_SOURCE)
add_definitions(-D_FILE_OFFSET_BITS=64)

set(LIBSRC lib/is_mounted.c lib/libcrc.c lib/libsort.c lib/libzdmwrap.c lib/zbc-ctrl.c)

add_library(zdm ${LIBSRC})

set (zdm_VERSION_MAJOR 0)
set (zdm_VERSION_MINOR 125)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/h/zdm_version.h.in"
  "${PROJECT_BINARY_DIR}/h/zdm_version.h"
  )

add_executable(zdmadm       src/zdmadm.c)
add_executable(zdm-rawio    src/zdm-rawio.c)
add_executable(zdm-mlog     src/zdm-mlog.c)
add_executable(zdm-status   src/zdm-status.c)
add_executable(zdm-zones    src/zdm-zones.c)
add_executable(zdm-report   src/zdm-report.c)
add_executable(zdm-zonecmd  src/zdm-zonecmd.c)
add_executable(blkreport    src/blkreport.c)
add_executable(blkreset     src/blkreset.c)


target_link_libraries (zdmadm       zdm uuid ssl sgutils2 udev)
target_link_libraries (zdm-rawio    zdm uuid ssl sgutils2)
target_link_libraries (zdm-mlog     zdm uuid ssl sgutils2)
target_link_libraries (zdm-status   zdm uuid ssl sgutils2)
target_link_libraries (zdm-zones    zdm uuid ssl sgutils2)
target_link_libraries (zdm-report   zdm uuid ssl sgutils2)
target_link_libraries (zdm-zonecmd  zdm uuid ssl sgutils2)
target_link_libraries (blkreport    zdm uuid ssl sgutils2)
target_link_libraries (blkreset     zdm uuid ssl sgutils2)

install(TARGETS zdmadm zdm-rawio zdm-mlog zdm-status zdm-zones zdm
                zdm-report zdm-zonecmd
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

set(INCFILES
        h/dm-zdm.h
        h/is_mounted.h
        h/libcrc.h
        h/libsort.h
        h/libzdmwrap.h
        h/libzdm-compat.h
        h/libzdm.h
        h/list.h
        h/utypes.h
        h/zbc-ctrl.h
        h/zdmioctl.h
        "${PROJECT_BINARY_DIR}/h/zdm_version.h"
)
install(FILES ${INCFILES} DESTINATION include/zdm)
