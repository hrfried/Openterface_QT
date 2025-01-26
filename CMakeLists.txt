cmake_minimum_required(VERSION 3.16)
project(
  openterfaceQT
  VERSION 1.0
  LANGUAGES C CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOUIC ON)

include(cmake/OpenterfaceUtils.cmake)

find_package(QT NAMES Qt5 Qt6 REQUIRED COMPONENTS Core)
find_package(
  Qt${QT_VERSION_MAJOR} REQUIRED
  COMPONENTS Concurrent
             Gui
             Multimedia
             MultimediaWidgets
             Network
             SerialPort
             Svg)
find_package(Qt${QT_VERSION_MAJOR} OPTIONAL_COMPONENTS Widgets)


# must go before the project()/enable_language() commands
openterface_update(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "Configs" FORCE)
if(NOT DEFINED CMAKE_BUILD_TYPE
    AND NOT OPENTERFACE_SKIP_DEFAULT_BUILD_TYPE
)
  message(STATUS "'Release' build type is used by default. Use CMAKE_BUILD_TYPE to specify build type (Release or Debug)")
  set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build")
endif()
if(DEFINED CMAKE_BUILD_TYPE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "${CMAKE_CONFIGURATION_TYPES}")
endif()

# --------------------------------
# Openterface Build Options
# --------------------------------
OPENTERFACE_OPTION(BUILD_WITH_DEBUG_INFO    "Include debug info into release binaries ('OFF' means default settings)" OFF )
OPENTERFACE_OPTION(ENABLE_CCACHE              "Use ccache"                                               (UNIX AND (CMAKE_GENERATOR MATCHES "Makefile" OR CMAKE_GENERATOR MATCHES "Ninja" OR CMAKE_GENERATOR MATCHES "Xcode")) )
# --------------------------------
# End Openterface Build Options
# --------------------------------


qt_standard_project_setup()

qt_add_executable(
  openterfaceQT
  WIN32
  MACOSX_BUNDLE
  global.h
  host/HostManager.cpp
  host/HostManager.h
  host/audiomanager.cpp
  host/audiomanager.h
  host/audiothread.cpp
  host/audiothread.h
  host/cameramanager.cpp
  host/cameramanager.h
  host/usbcontrol.cpp
  host/usbcontrol.h
  main.cpp
  resources/version.h
  scripts/KeyboardMouse.cpp
  scripts/KeyboardMouse.h
  scripts/Lexer.cpp
  scripts/Lexer.h
  scripts/Parser.cpp
  scripts/Parser.h
  scripts/semanticAnalyzer.cpp
  scripts/semanticAnalyzer.h
  serial/SerialPortManager.cpp
  serial/SerialPortManager.h
  serial/ch9329.h
  target/KeyboardManager.cpp
  target/KeyboardManager.h
  target/Keymapping.h
  target/MouseManager.cpp
  target/MouseManager.h
  target/mouseeventdto.cpp
  target/mouseeventdto.h
  ui/TaskManager.cpp
  ui/TaskManager.h
  ui/audiopage.cpp
  ui/audiopage.h
  ui/cameraajust.cpp
  ui/cameraajust.h
  ui/fpsspinbox.cpp
  ui/fpsspinbox.h
  ui/globalsetting.cpp
  ui/globalsetting.h
  ui/hardwarepage.cpp
  ui/hardwarepage.h
  ui/helppane.cpp
  ui/helppane.h
  ui/inputhandler.cpp
  ui/inputhandler.h
  ui/loghandler.cpp
  ui/loghandler.h
  ui/logpage.cpp
  ui/logpage.h
  ui/mainwindow.cpp
  ui/mainwindow.h
  ui/mainwindow.ui
  ui/metadatadialog.cpp
  ui/metadatadialog.h
  ui/scripttool.cpp
  ui/scripttool.h
  ui/serialportdebugdialog.cpp
  ui/serialportdebugdialog.h
  ui/settingdialog.cpp
  ui/settingdialog.h
  ui/settingdialog.ui
  ui/statusbarmanager.cpp
  ui/statusbarmanager.h
  ui/statusevents.h
  ui/statuswidget.cpp
  ui/statuswidget.h
  ui/toggleswitch.cpp
  ui/toggleswitch.h
  ui/toolbarmanager.cpp
  ui/toolbarmanager.h
  ui/versioninfomanager.cpp
  ui/versioninfomanager.h
  ui/videopage.cpp
  ui/videopage.h
  ui/videopane.cpp
  ui/videopane.h
  video/videohid.cpp
  video/videohid.h)
target_link_libraries(
  openterfaceQT
  PRIVATE Qt::Concurrent
          Qt::Core
          Qt::Gui
          Qt::Multimedia
          Qt::MultimediaWidgets
          Qt::Network
          Qt::SerialPort
          Qt::Svg)

# Resources:
set(mainwindow_resource_files
    "ui/../images/capture.svg"
    "ui/../images/content_dark_eng.png"
    "ui/../images/contrast.svg"
    "ui/../images/icon_128.ico"
    "ui/../images/icon_128.png"
    "ui/../images/icon_32.ico"
    "ui/../images/icon_32.png"
    "ui/../images/icon_64.ico"
    "ui/../images/icon_64.png"
    "ui/../images/keyboard-down.svg"
    "ui/../images/keyboard-pressed.svg"
    "ui/../images/keyboard-up.svg"
    "ui/../images/keyboard.svg"
    "ui/../images/mouse-default.svg"
    "ui/../images/mouse-left-button.svg"
    "ui/../images/mouse-middle-button.svg"
    "ui/../images/mouse-right-button.svg"
    "ui/../images/paste.svg"
    "ui/../images/screensaver.svg"
    "ui/../images/shutter.svg"
    "ui/../images/zoom_in.svg"
    "ui/../images/zoom_out.svg"
    "ui/../images/zoom_reduction.svg")

qt_add_resources(
  openterfaceQT
  "mainwindow"
  PREFIX
  "/"
  BASE
  "ui"
  FILES
  ${mainwindow_resource_files})
set(qmake_immediate_resource_files "openterfaceQT.rc")

qt_add_resources(openterfaceQT "qmake_immediate" PREFIX "/" FILES
                 ${qmake_immediate_resource_files})

if((QT_VERSION_MAJOR GREATER 4))
  target_link_libraries(openterfaceQT PRIVATE Qt::Widgets)
endif()

if(WIN32)
  target_include_directories(openterfaceQT PRIVATE lib)

  target_link_libraries(
    openterfaceQT PRIVATE # Remove: L${CMAKE_CURRENT_SOURCE_DIR}/lib
                          hid libusb-1.0 setupapi)
endif()

if(UNIX)
  target_include_directories(openterfaceQT PRIVATE /usr/include)

  target_link_libraries(openterfaceQT PRIVATE usb-1.0)
endif()

install(
  TARGETS openterfaceQT
  BUNDLE DESTINATION .
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

qt_generate_deploy_app_script(TARGET openterfaceQT OUTPUT_SCRIPT deploy_script
                              NO_UNSUPPORTED_PLATFORM_ERROR)
install(SCRIPT ${deploy_script})
