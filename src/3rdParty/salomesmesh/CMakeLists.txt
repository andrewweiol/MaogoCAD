
set(CMAKE_AUTOMOC FALSE)

SET(SMESH_VERSION_MAJOR 7)  # see main CMakeLists.txt of FreeCAD
SET(SMESH_VERSION_MINOR 7)
SET(SMESH_VERSION_PATCH 1)
SET(SMESH_VERSION_TWEAK 0)

# Suppress gcc warnings
if(CMAKE_COMPILER_IS_GNUCXX)
    set(CMAKE_CXX_FLAGS " ${CMAKE_CXX_FLAGS} -Wno-sign-compare -Wno-reorder -Wno-switch -Wno-unused-variable -Wno-unused-but-set-variable -Wno-comment -Wno-unused-parameter -Wno-empty-body -Wno-pedantic")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-variable -Wno-unused-but-set-variable")

    unset(_flag_found CACHE)
    check_cxx_compiler_flag("-Wno-unused-result" _flag_found)
    if (_flag_found)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-result")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-unused-result")
    endif ()

    unset(_flag_found CACHE)
    check_cxx_compiler_flag("-Wno-cast-function-type" _flag_found)
    if (_flag_found)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-cast-function-type")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-cast-function-type")
    endif ()

    unset(_flag_found CACHE)
    check_cxx_compiler_flag("-Wno-maybe-uninitialized" _flag_found)
    if (_flag_found)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-maybe-uninitialized")
    endif ()

    unset(_flag_found CACHE)
    check_cxx_compiler_flag("-Wno-missing-field-initializers" _flag_found)
    if (_flag_found)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-missing-field-initializers")
    endif ()

# Suppress clang warnings
elseif(CMAKE_COMPILER_IS_CLANGXX)
    set(CMAKE_CXX_FLAGS " ${CMAKE_CXX_FLAGS} -Wno-self-assign -Wno-sign-compare -Wno-logical-op-parentheses -Wno-reorder -Wno-switch -Wno-switch-enum -Wno-unknown-pragmas -Wno-unused-variable -Wno-unused-private-field")
    set(CMAKE_CXX_FLAGS " ${CMAKE_CXX_FLAGS} -Wno-unused-function -Wno-sometimes-uninitialized -Wno-overloaded-virtual -Wno-dynamic-class-memaccess -Wno-comment -Wno-unused-parameter -Wno-extra-semi")

    unset(_flag_found CACHE)
    check_cxx_compiler_flag("-Wno-deprecated-copy" _flag_found)
    if (_flag_found)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-deprecated-copy")
    endif ()

    unset(_flag_found CACHE)
    check_cxx_compiler_flag("-Wno-missing-field-initializers" _flag_found)
    if (_flag_found)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-missing-field-initializers")
    endif ()

    unset(_flag_found CACHE)
    check_cxx_compiler_flag("-Wno-c++17-extensions" _flag_found)
    if (_flag_found)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-c++17-extensions")
    endif ()
endif()


if (VTK_OPTIONS)
    add_definitions(${VTK_OPTIONS})
endif()

include_directories(
    src/SMDS
    src/Driver
    src/DriverUNV
    src/DriverDAT
    src/DriverSTL
    src/StdMeshers
    inc
    ${CMAKE_BINARY_DIR}/src
    ${CMAKE_SOURCE_DIR}/src
    ${CMAKE_CURRENT_BINARY_DIR}
    ${Boost_INCLUDE_DIRS}
    ${OCC_INCLUDE_DIR}
    ${NGLIB_INCLUDE_DIR}
    ${NETGEN_INCLUDE_DIRS}
    ${ZLIB_INCLUDE_DIR}
    ${VTK_INCLUDE_DIRS}
    ${HDF5_INCLUDE_DIR}
    ${MEDFILE_INCLUDE_DIRS}
    ${BOOST_INCLUDE_DIR}
)
link_directories(${OCC_LIBRARY_DIR})

if(MSVC)
    if(BUILD_FEM_NETGEN)
        set(SMESH_LIBS
            debug     MSVCRTD.LIB
            debug     MSVCPRTD.LIB
            optimized MSVCRT.LIB
            optimized MSVCPRT.LIB
            Rpcrt4.lib
            ${NGLIB_LIBRARIES}
            ${NGLIB_DEBUG_LIBRARIES}
            ${OCC_LIBRARIES}
            ${OCC_DEBUG_LIBRARIES}
            ${OCC_OCAF_DEBUG_LIBRARIES}
            ${OCC_OCAF_LIBRARIES}
        )
        ADD_DEFINITIONS(${NETGEN_DEFINITIONS})
    else(BUILD_FEM_NETGEN)
        set(SMESH_LIBS
            debug     MSVCRTD.LIB
            debug     MSVCPRTD.LIB
            optimized MSVCRT.LIB
            optimized MSVCPRT.LIB
            Rpcrt4.lib
            ${OCC_LIBRARIES}
            ${OCC_DEBUG_LIBRARIES}
            ${OCC_OCAF_DEBUG_LIBRARIES}
            ${OCC_OCAF_LIBRARIES}
        )
    endif(BUILD_FEM_NETGEN)

    #Universal C runtime introduced in VS 2015 (cl version 19)
    if (NOT (CMAKE_CXX_COMPILER_VERSION VERSION_LESS "19"))
        list(APPEND SMESH_LIBS
            debug vcruntimed.lib
            debug ucrtd.lib
            optimized vcruntime.lib
            optimized ucrt.lib
        )
    endif()
else(MSVC)
    if(BUILD_FEM_NETGEN)
        set(SMESH_LIBS
            ${NGLIB_LIBRARIES}
            ${OCC_LIBRARIES}
            ${OCC_OCAF_LIBRARIES}
        )
        ADD_DEFINITIONS(${NETGEN_DEFINITIONS})
    else(BUILD_FEM_NETGEN)
        set(SMESH_LIBS
            ${OCC_LIBRARIES}
            ${OCC_OCAF_LIBRARIES}
        )
    endif(BUILD_FEM_NETGEN)
endif(MSVC)

SET(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)


############################
# Check 32/64 bit platform #
############################
IF (${CMAKE_SIZEOF_VOID_P} MATCHES "8") # It is 64bit, 32 bit systems returns 4
    ADD_DEFINITIONS(-D_OCC64)
    IF(CMAKE_COMPILER_IS_GNUCXX)
    # 0001715: Mefisto Mesh Fails since translated by f2c
    #    ADD_DEFINITIONS(-DPCLINUX64)
    ENDIF(CMAKE_COMPILER_IS_GNUCXX)
    #MESSAGE("-- Build 64bit")
ELSE (${CMAKE_SIZEOF_VOID_P} MATCHES "8")
    #MESSAGE("-- Build 32bit")
ENDIF(${CMAKE_SIZEOF_VOID_P} MATCHES "8")

######################################################
# Configure Build settings according to the platform #
######################################################
IF(UNIX)
    # Same settings are used for both MacOSX and Unix/Linux
    ADD_DEFINITIONS(-DHAVE_CONFIG_H -DHAVE_LIMITS_H -DCSFDB -DLIN -DOCC_CONVERT_SIGNALS)
ELSE(UNIX)
  IF(WIN32)
    ######### Windows ###########
    IF(MSVC)
        ADD_DEFINITIONS(-DWNT -DWIN32 -D_WINDOWS -DCSFDB -DUSE_CLOCK -DMSDOS -DNO_ONEXIT -DNO_My_ctype -DNO_ISATTY -DNO_FPINIT /wd4290 /wd4251 /wd4018 /wd4800 /wd4996 /wd4244 /wd4805 /wd4806 /wd4275 /wd4005 /wd4099 /wd4101 /wd4146 /wd4267 /wd4390 /wd4503 /wd4436)
    ELSE(MSVC)
        ADD_DEFINITIONS(-DWNT -DWIN32 -D_WINDOWS -DCSFDB)
    ENDIF(MSVC)
  ELSE(WIN32)
    MESSAGE("Unknown platform")
  ENDIF(WIN32)
ENDIF(UNIX)

#########
# SMDS #
#########
FILE(GLOB SMDS_source_files src/SMDS/*.cpp inc/SMDS*.hxx)
ADD_LIBRARY(SMDS SHARED ${SMDS_source_files})
TARGET_LINK_LIBRARIES(SMDS ${SMESH_LIBS} ${HDF5_C_STATIC_LIBRARY} ${VTK_LIBRARIES})

SET_BIN_DIR(SMDS SMDS)
if(WIN32)
    set_target_properties(SMDS PROPERTIES COMPILE_FLAGS "-DSMDS_EXPORTS -DUTILS_EXPORTS")
endif(WIN32)


##########
# Driver #
##########
FILE(GLOB Driver_source_files src/Driver/*.cpp inc/Driver_*.hxx)
ADD_LIBRARY(Driver SHARED ${Driver_source_files})
TARGET_LINK_LIBRARIES(Driver ${SMESH_LIBS})

SET_BIN_DIR(Driver Driver)
if(WIN32)
    set_target_properties(Driver PROPERTIES COMPILE_FLAGS "-DMESHDRIVER_EXPORTS -DSMESHUtils_EXPORTS")
endif(WIN32)

#############
# DriverSTL #
#############
FILE(GLOB DriverSTL_source_files src/DriverSTL/*.cpp inc/DriverSTL*.h*)
ADD_LIBRARY(DriverSTL SHARED ${DriverSTL_source_files})
TARGET_LINK_LIBRARIES(DriverSTL ${SMESH_LIBS} Driver SMDS ${Boost_LIBRARIES})

SET_BIN_DIR(DriverSTL DriverSTL)
if(WIN32)
    set_target_properties(DriverSTL PROPERTIES COMPILE_FLAGS "-DMESHDRIVERSTL_EXPORTS -DSMESHUtils_EXPORTS -DBASICS_EXPORTS")
endif(WIN32)

#############
# DriverDAT #
#############
FILE(GLOB DriverDAT_source_files src/DriverDAT/*.cpp inc/DriverDAT*.h*)
ADD_LIBRARY(DriverDAT SHARED ${DriverDAT_source_files})
TARGET_LINK_LIBRARIES(DriverDAT ${SMESH_LIBS} SMDS Driver)

SET_BIN_DIR(DriverDAT DriverDAT)
if(WIN32)
    set_target_properties(DriverDAT PROPERTIES COMPILE_FLAGS "-DMESHDRIVERDAT_EXPORTS")
endif(WIN32)

#############
# DriverUNV #
#############
FILE(GLOB DriverUNV_source_files src/DriverUNV/*.cpp inc/DriverUNV*.h*)
ADD_LIBRARY(DriverUNV SHARED ${DriverUNV_source_files})
TARGET_LINK_LIBRARIES(DriverUNV ${SMESH_LIBS} SMESHDS SMDS Driver)

SET_BIN_DIR(DriverUNV DriverUNV)
if(WIN32)
    set_target_properties(DriverUNV PROPERTIES COMPILE_FLAGS "-DMESHDRIVERUNV_EXPORTS")
endif(WIN32)

###########
# SMESHDS #
###########
FILE(GLOB SMESHDS_source_files src/SMESHDS/*.cpp inc/SMESHDS_*.h*)
ADD_LIBRARY(SMESHDS SHARED ${SMESHDS_source_files})
TARGET_LINK_LIBRARIES(SMESHDS ${SMESH_LIBS} SMDS ${Boost_LIBRARIES} )

SET_BIN_DIR(SMESHDS SMESHDS)

#########
# SMESH #
#########
FILE(GLOB SMESH_source_files src/SMESH/*.c src/SMESH/*.cpp src/Controls/*.cpp inc/SMESH_*.h*)
ADD_LIBRARY(SMESH SHARED ${SMESH_source_files})
TARGET_LINK_LIBRARIES(SMESH SMDS SMESHDS  Driver DriverSTL DriverDAT DriverUNV ${SMESH_LIBS} ${OPENCASCADE_LIBRARIES} ${OCC_LIBRARIES} ${OCC_DEBUG_LIBRARIES} ${OCC_OCAF_DEBUG_LIBRARIES} ${OCC_OCAF_LIBRARIES} ${MEDFILE_LIBRARIES} ${VTK_LIBRARIES})
 
SET_BIN_DIR(SMESH SMESH)
if(WIN32)
    set_target_properties(SMESH PROPERTIES COMPILE_FLAGS "-DSMESH_EXPORTS -DSMESHCONTROLS_EXPORTS -DSMESHimpl_EXPORTS -DMESHDriverGMF_EXPORTS -DMESHDRIVERMED_EXPORTS -DMEDWRAPPER_EXPORTS -DMEDWRAPPER_BASE_EXPORTS -DMEDWRAPPER_FACTORY_EXPORTS -DSMESHUtils_EXPORTS -DUTILS_EXPORTS -DMEDWRAPPER_V2_2_EXPORTS")
endif(WIN32)

##############
# MEFISTO2   #
##############
set(MEFISTO2_source_files
	src/MEFISTO2/aptrte.cpp
        src/MEFISTO2/trte.c
        inc/Rn.h
        inc/aptrte.h
)        

ADD_LIBRARY(MEFISTO2 SHARED ${MEFISTO2_source_files})

SET_BIN_DIR(MEFISTO2 MEFISTO2)
TARGET_LINK_LIBRARIES(MEFISTO2 ${SMESH_LIBS}) # in case of Windows link the prebuild Fortran lib
set_target_properties(MEFISTO2 PROPERTIES COMPILE_FLAGS "-DMEFISTO2D_EXPORTS")


##############
# StdMeshers #
##############
FILE(GLOB StdMeshers_source_files src/StdMeshers/*.cpp inc/StdMeshers_*.h*)
ADD_LIBRARY(StdMeshers SHARED ${StdMeshers_source_files})
TARGET_LINK_LIBRARIES(StdMeshers SMESH ${SMESH_LIBS} MEFISTO2)

SET_BIN_DIR(StdMeshers StdMeshers)
if(WIN32)
    set_target_properties(StdMeshers PROPERTIES COMPILE_FLAGS "-DSTDMESHERS_EXPORTS -DSMESHUtils_EXPORTS")
else(WIN32)
    set_target_properties(StdMeshers PROPERTIES COMPILE_FLAGS "${StdMeshers_CFLAGS}")
endif(WIN32)

# Disable for all other compilers but MSVC
if (BUILD_FEM_NETGEN)

################
# NETGENPlugin #
################
FILE(GLOB NETGENPlugin_source_files src/NETGENPlugin/*.cpp inc/NETGENPlugin_*.h* )
ADD_LIBRARY(NETGENPlugin SHARED ${NETGENPlugin_source_files})
TARGET_LINK_LIBRARIES(NETGENPlugin SMDS SMESHDS SMESH StdMeshers ${SMESH_LIBS} )

SET_BIN_DIR(NETGENPlugin NETGENPlugin)
if(WIN32)
    set_target_properties(NETGENPlugin PROPERTIES COMPILE_FLAGS "-DNETGENPLUGIN_EXPORTS -DNO_PARALLEL_THREADS -DOCCGEOMETRY -DNETGEN_VERSION=${NETGEN_VERSION_C}")
else()
    set_target_properties(NETGENPlugin PROPERTIES COMPILE_FLAGS "${NETGEN_CXX_FLAGS}")
endif(WIN32)

    set_target_properties(NETGENPlugin PROPERTIES CXX_STANDARD_REQUIRED ON)
    set_target_properties(NETGENPlugin PROPERTIES CXX_STANDARD 17) # Standard std=c++1y could work as well
endif (BUILD_FEM_NETGEN)

if(WIN32)
    INSTALL(TARGETS SMDS Driver DriverSTL DriverDAT DriverUNV SMESHDS SMESH MEFISTO2 StdMeshers
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
else(WIN32)
    INSTALL(TARGETS SMDS Driver DriverSTL DriverDAT DriverUNV SMESHDS SMESH MEFISTO2 StdMeshers
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
endif(WIN32)

if (BUILD_FEM_NETGEN)
    if(WIN32)
        INSTALL(TARGETS NETGENPlugin
            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        )
    else(WIN32)
        INSTALL(TARGETS NETGENPlugin
            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        )
    endif(WIN32)
endif (BUILD_FEM_NETGEN)

