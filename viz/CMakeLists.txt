project(mars_viz)
set(PROJECT_VERSION 1.0)
set(PROJECT_DESCRIPTION "This is ...")
cmake_minimum_required(VERSION 2.6)

include(FindPkgConfig)

find_package(lib_manager)
lib_defaults()
define_module_info()

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR} "${PROJECT_SOURCE_DIR}/cmake")

MACRO(CMAKE_USE_FULL_RPATH install_rpath)
    SET(CMAKE_SKIP_BUILD_RPATH  FALSE)
    SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)
    SET(CMAKE_INSTALL_RPATH ${install_rpath})
    SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
ENDMACRO(CMAKE_USE_FULL_RPATH)
CMAKE_USE_FULL_RPATH("${CMAKE_INSTALL_PREFIX}/lib")

pkg_check_modules(PKGCONFIG
  lib_manager
  mars_interfaces
  mars_graphics
  cfg_manager
  mars_scene_loader
  main_gui
  configmaps
  data_broker
)
include_directories(${PKGCONFIG_INCLUDE_DIRS})
link_directories(${PKGCONFIG_LIBRARY_DIRS})
add_definitions(${PKGCONFIG_CLFAGS_OTHER})  #flags excluding the ones with -I

setup_qt()

set(SOURCES
    src/Viz.cpp
    src/GraphicsTimer.cpp
)

set(QT_MOC_HEADER
    src/GraphicsTimer.h
)

if (${USE_QT5})
qt5_wrap_cpp ( QT_MOC_HEADER_SRC ${QT_MOC_HEADER} )
else (${USE_QT5})
qt4_wrap_cpp ( QT_MOC_HEADER_SRC ${QT_MOC_HEADER} )
endif (${USE_QT5})

set(TARGET_SRC
    ${QT_MOC_HEADER_SRC}
    ${SOURCES}
)

IF( MINGW )
  # resource compilation for MinGW
  ADD_CUSTOM_COMMAND( OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/mars_icon.o
                      COMMAND windres.exe -I${CMAKE_CURRENT_SOURCE_DIR} -i${CMAKE_CURRENT_SOURCE_DIR}/resources/appicon.rc
                             -o ${CMAKE_CURRENT_BINARY_DIR}/mars_icon.o )
  SET(TARGET_SRC ${TARGET_SRC} ${CMAKE_CURRENT_BINARY_DIR}/mars_icon.o)
#ELSE( MINGW )
#  SET(TARGET_SRC ${TARGET_SRC} resources/mars_icon.ico)
ENDIF( MINGW )

add_executable(${PROJECT_NAME}_app src/main.cpp ${TARGET_SRC})
add_library(${PROJECT_NAME} SHARED ${TARGET_SRC})

if (${USE_QT5})
qt5_use_modules(${PROJECT_NAME} Widgets)
qt5_use_modules(mars_viz Widgets)
endif (${USE_QT5})

IF (WIN32)
  set(WIN_LIBS
        -lwinmm
  )
  SET_TARGET_PROPERTIES(${PROJECT_NAME} PROPERTIES LINK_FLAGS -Wl,--stack,0x1000000)
ENDIF (WIN32)

target_link_libraries(${PROJECT_NAME}
            ${QT_LIBRARIES}
            ${QT_QTOPENGL_LIBRARIES}
            ${QT_QTXML_LIBRARY}
            ${WIN_LIBS}
            ${PKGCONFIG_LIBRARIES}
)

target_link_libraries(${PROJECT_NAME}_app mars_viz)

INSTALL(TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_app
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib)

IF (WIN32)
  SET (POSTLIB ".dll")
  SET (PRELIB "lib")
  SET (LIBPATH "bin")
ELSE (WIN32)
  IF (APPLE)
    SET (POSTLIB ".dylib")
    SET (PRELIB "lib")
    SET (LIBPATH "lib")
  ELSE (APPLE)
    SET (POSTLIB ".so")
    SET (PRELIB "lib")
    SET (LIBPATH "lib")
  ENDIF (APPLE)
ENDIF (WIN32)

set(ADDITIONAL_LIBS "" CACHE INTERNAL "")


configure_file(mars_viz.pc.in ${CMAKE_BINARY_DIR}/mars_viz.pc @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/mars_viz.pc DESTINATION lib/pkgconfig/)
install(FILES ${CMAKE_SOURCE_DIR}/src/Viz.h ${CMAKE_SOURCE_DIR}/src/GraphicsTimer.h ${CMAKE_SOURCE_DIR}/src/MyApp.h DESTINATION include/mars/viz/)
