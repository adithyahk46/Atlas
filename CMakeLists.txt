# Modernized CMakeLists.txt for Atlas (Upgraded from ~2014 version)
# Assumes CMake 3.20+, Qt5 (or Qt6), OSG 3.8+, OSGEarth 3.x

cmake_minimum_required(VERSION 3.20)

project(Atlas LANGUAGES CXX)

# Options
option(PRELOAD_MAP "Preload default map." ON)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Build type default
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# Paths
set(DIST_PATH ${PROJECT_BINARY_DIR}/dist)
set(DIST_PLUGINS_PATH ${DIST_PATH}/plugins)
set(DIST_LIB_PATH ${DIST_PATH}/lib)
set(DIST_RESOURCES_PATH ${DIST_PATH}/resources)


# Module path
#set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/CMakeModules;${CMAKE_MODULE_PATH}")

# Qt setup (Qt5; for Qt6, replace with qt6_standard_project_setup() and Qt6:: components)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)  # Modern alternative to QT5_WRAP_UI
set(CMAKE_AUTOUIC_SEARCH_PATHS
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/core
    ${CMAKE_CURRENT_SOURCE_DIR}/core/AtlasMainWindow
    ${CMAKE_CURRENT_SOURCE_DIR}/core/DataManager
)
set(QT_MIN_VERSION 5.15.0)  # Updated from 5.11

# Output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${DIST_PATH})


set(CMAKE_TOOLCHAIN_FILE "D:/vcpkg/scripts/buildsystems/vcpkg.cmake" CACHE STRING "Vcpkg toolchain file")
set(CMAKE_PREFIX_PATH "D:/vcpkg/installed/x64-windows;D:/github/osgearth/install/osgearth_lib;C:/Qt/5.15.18/msvc2019_64")



# Dependencies
find_package(Qt5 ${QT_MIN_VERSION} COMPONENTS Core Gui Widgets Svg OpenGL Network Xml REQUIRED)
find_package(Threads REQUIRED)  # For threading
find_package(OpenSceneGraph 3.6 REQUIRED COMPONENTS osgViewer osgGA osgDB osgManipulator osgSim osgParticle osgText osgUtil)
find_package(osgEarth CONFIG REQUIRED)
find_package(GDAL REQUIRED)
# find_package(FriBiDi)  # Uncomment if needed

include_directories(
    ${PROJECT_BINARY_DIR}
    ${OSG_INCLUDE_DIR}
    ${GDAL_INCLUDE_DIR}
    ${OSGEARTH_INCLUDE_DIRS}
    ${OSGEARTH_INCLUDE_DIR}
    core
)

#file(GLOB_RECURSE HFiles RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)
#file(GLOB_RECURSE CppFiles RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
#file(GLOB_RECURSE UiFiles RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/src/*.ui)

set(${PROJECT_NAME}_HDRS
    Atlas/Atlas.h
    Atlas/AtlasSplashScreen.h
    Atlas/resource.h
    core/AtlasMainWindow/atlasmainwindow_global.h
    core/AtlasMainWindow/AtlasMainWindow.h
    core/AtlasMainWindow/NXDockWidget.h
    core/AtlasMainWindow/NXDockWidgetTabBar.h
    core/AtlasMainWindow/NXDockWidgetTabButton.h
    core/AtlasMainWindow/NXDockWidgetTitle.h
    core/SettingsManager/SettingsManager_global.h
    core/SettingsManager/SettingsManager.h
    core/DataManager/FontVisitor.h
    core/DataManager/DataFormats.h
    core/DataManager/DataManager_global.h
    core/DataManager/DataManager.h
    core/DataManager/DataRecord.h
    core/DataManager/DataTree.h
    core/DataManager/FeatureStyleSettingDlg.h
    core/DataManager/FindNode.hpp
    core/MapController/MapController_global.h
    core/MapController/MapController.h
    core/PluginInterface/PluginInterface_global.h
    core/PluginInterface/PluginInterface.h
    core/PluginManager/PluginManager_global.h
    core/PluginManager/PluginManager.h
    core/MousePicker/MousePicker_global.h
    core/MousePicker/MousePicker.h
    core/ViewerWidget/Compass.h
    core/ViewerWidget/ViewerWidget_global.h
    core/ViewerWidget/ViewerWidget.h
    core/osgQt/GraphicsWindowQt.h
    )

set(${PROJECT_NAME}_SRCS
    Atlas/main.cpp
    Atlas/Atlas.cpp
    Atlas/AtlasSplashScreen.cpp
    core/AtlasMainWindow/AtlasMainWindow.cpp
    core/AtlasMainWindow/NXDockWidget.cpp
    core/AtlasMainWindow/NXDockWidgetTabBar.cpp
    core/AtlasMainWindow/NXDockWidgetTabButton.cpp
    core/AtlasMainWindow/NXDockWidgetTitle.cpp
    core/SettingsManager/SettingsManager.cpp
    core/DataManager/FontVisitor.cpp
    core/DataManager/DataManager.cpp
    core/DataManager/DataRecord.cpp
    core/DataManager/DataTree.cpp
    core/DataManager/FeatureStyleSettingDlg.cpp
    core/MapController/MapController.cpp
    core/PluginInterface/PluginInterface.cpp
    core/PluginManager/PluginManager.cpp
    core/MousePicker/MousePicker.cpp
    core/ViewerWidget/Compass.cpp
    core/ViewerWidget/ViewerWidget.cpp
    core/osgQt/GraphicsWindowQt.cpp

#    core/AtlasMainWindow/AtlasMainWindow.ui
#    core/DataManager/StyleTab.ui
#    core/DataManager/FeatureStyleSettingDlg.ui

    # Resources:
    resource.qrc
)

set (${PROJECT_NAME}_UIS
    core/AtlasMainWindow/AtlasMainWindow.ui
    core/DataManager/StyleTab.ui
    core/DataManager/FeatureStyleSettingDlg.ui
    )

#QT5_ADD_RESOURCES(RSCS ${enzarmonitor_RSCS})
#QT5_WRAP_UI(UIS ${${PROJECT_NAME}_UIS})

add_executable(${PROJECT_NAME}
    ${${PROJECT_NAME}_SRCS}
    ${${PROJECT_NAME}_HDRS}
    ${${PROJECT_NAME}_UIS}
)

if(Qt5_FOUND)
    message(STATUS "Qt5 found: ${Qt5_VERSION}")
    target_link_libraries(${PROJECT_NAME} PRIVATE
        Qt5::Core
        Qt5::Gui
        Qt5::Widgets
        Qt5::Svg
        Qt5::OpenGL
        Qt5::Network
        Qt5::Xml
    )
else()
    message(FATAL_ERROR "Qt5 not found. Please ensure it is installed and CMAKE_PREFIX_PATH is set correctly.")
endif()

if(OpenSceneGraph_FOUND AND NOT TARGET OpenSceneGraph::OpenSceneGraph)
    add_library(OpenSceneGraph::OpenSceneGraph INTERFACE IMPORTED)

    set_target_properties(OpenSceneGraph::OpenSceneGraph PROPERTIES
        INTERFACE_INCLUDE_DIRECTORIES "${OPENSCENEGRAPH_INCLUDE_DIRS}"
        INTERFACE_LINK_LIBRARIES "${OPENSCENEGRAPH_LIBRARIES}"
    )
    message(STATUS "OpenSceneGraph found: ${OPENSCENEGRAPH_VERSION} : ${OPENSCENEGRAPH_INCLUDE_DIRS} ")
    target_link_libraries(${PROJECT_NAME} PRIVATE
    OpenSceneGraph::OpenSceneGraph
    )
else()
    message(FATAL_ERROR "OpenSceneGraph not found. Please ensure it is installed and CMAKE_PREFIX_PATH is set correctly.")
endif()

if(osgEarth_FOUND)
    message(STATUS "osgEarth found: ${OSGEARTH_VERSION}")
    target_link_libraries(${PROJECT_NAME} PRIVATE osgEarth::osgEarth)
else()
    message(FATAL_ERROR "osgEarth not found. Please ensure it is installed and CMAKE_PREFIX_PATH is set correctly.")
endif() 

if(GDAL_FOUND)
    message(STATUS "GDAL found: ${GDAL_VERSION}")
    target_link_libraries(${PROJECT_NAME} PRIVATE GDAL::GDAL)
else()
    message(FATAL_ERROR "GDAL not found. Please ensure it is installed and CMAKE_PREFIX_PATH is set correctly.")
endif()

# Linking (use imported targets)
#target_link_libraries(${PROJECT_NAME} PRIVATE
#    Threads::Threads
#    # Add OSGQt if available: find_package(osgQt REQUIRED); target_link_libraries(... osgQt::osgQt)
#)

# Defining plugins.
function(DefinePlugin pluginName)
  message("Defining plugin: ${pluginName}")
  option (PLUGIN_${pluginName} "Use ${pluginName}")
  if (PLUGIN_${pluginName})
    add_subdirectory(plugins/${pluginName})
    include_directories(plugins/${pluginName}/include)
    string(TOUPPER ${pluginName} upperName)
    add_definitions(-D${upperName})
  endif ()
endfunction()

macro(SUBDIRLIST result curdir)
  file(GLOB children RELATIVE ${curdir} ${curdir}/*)
  set(dirlist "")
  foreach(child ${children})
    if(IS_DIRECTORY ${curdir}/${child})
      list(APPEND dirlist ${child})
    endif()
  endforeach()
  set(${result} ${dirlist})
endmacro()

SUBDIRLIST(PLUGINS ${CMAKE_CURRENT_SOURCE_DIR}/plugins)

foreach(p ${PLUGINS})
  DefinePlugin(${p})
endforeach()

# Cleaning all previously built plugins.
file(REMOVE_RECURSE ${DIST_PLUGINS_PATH})

# Copying resources.
file(GLOB resdata resources/*)
file(COPY ${resdata} DESTINATION ${DIST_RESOURCES_PATH})


add_subdirectory(libs)

install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION bin)