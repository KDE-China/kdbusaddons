cmake_minimum_required(VERSION 3.0)

set(KF5_VERSION "5.50.0") # handled by release scripts
project(KDBusAddons VERSION ${KF5_VERSION})

include(FeatureSummary)
find_package(ECM 5.49.0  NO_MODULE)
set_package_properties(ECM PROPERTIES TYPE REQUIRED DESCRIPTION "Extra CMake Modules." URL "https://projects.kde.org/projects/kdesupport/extra-cmake-modules")
feature_summary(WHAT REQUIRED_PACKAGES_NOT_FOUND FATAL_ON_MISSING_REQUIRED_PACKAGES)


set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${ECM_KDE_MODULE_DIR})

include(KDEInstallDirs)
include(KDEFrameworkCompilerSettings NO_POLICY_SCOPE)
include(KDECMakeSettings)
include(ECMQtDeclareLoggingCategory)

set(REQUIRED_QT_VERSION 5.6.0)
find_package(Qt5DBus ${REQUIRED_QT_VERSION} REQUIRED NO_MODULE)
find_package(Qt5X11Extras ${REQUIRED_QT_VERSION} NO_MODULE)


include(GenerateExportHeader)
include(CMakePackageConfigHelpers)
include(ECMSetupVersion)
include(ECMGenerateHeaders)
include(ECMMarkNonGuiExecutable)
include(ECMAddQch)

include(ECMPoQmTools)

ecm_setup_version(PROJECT VARIABLE_PREFIX KDBUSADDONS
                        VERSION_HEADER "${CMAKE_CURRENT_BINARY_DIR}/kdbusaddons_version.h"
                        PACKAGE_VERSION_FILE "${CMAKE_CURRENT_BINARY_DIR}/KF5DBusAddonsConfigVersion.cmake"
                        SOVERSION 5)

if(Qt5X11Extras_FOUND)
    set(qtx11extras Qt5::X11Extras)
    set(HAVE_X11 TRUE)
else()
    set(HAVE_X11 FALSE)
endif()

option(BUILD_QCH "Build API documentation in QCH format (for e.g. Qt Assistant, Qt Creator & KDevelop)" OFF)
add_feature_info(QCH ${BUILD_QCH} "API documentation in QCH format (for e.g. Qt Assistant, Qt Creator & KDevelop)")

if (IS_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/po")
    ecm_install_po_files_as_qm(po)
endif()

add_subdirectory(src)
if (BUILD_TESTING)
    add_subdirectory(autotests)
    add_subdirectory(tests)
endif()

# create a Config.cmake and a ConfigVersion.cmake file and install them
set(CMAKECONFIG_INSTALL_DIR "${KDE_INSTALL_CMAKEPACKAGEDIR}/KF5DBusAddons")

if (BUILD_QCH)
    ecm_install_qch_export(
        TARGETS KF5DBusAddons_QCH
        FILE KF5DBusAddonsQchTargets.cmake
        DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
        COMPONENT Devel
    )
    set(PACKAGE_INCLUDE_QCHTARGETS "include(\"\${CMAKE_CURRENT_LIST_DIR}/KF5DBusAddonsQchTargets.cmake\")")
endif()

configure_package_config_file("${CMAKE_CURRENT_SOURCE_DIR}/KF5DBusAddonsConfig.cmake.in"
                              "${CMAKE_CURRENT_BINARY_DIR}/KF5DBusAddonsConfig.cmake"
                              INSTALL_DESTINATION  ${CMAKECONFIG_INSTALL_DIR}
                              )

install(FILES  "${CMAKE_CURRENT_BINARY_DIR}/KF5DBusAddonsConfig.cmake"
               "${CMAKE_CURRENT_BINARY_DIR}/KF5DBusAddonsConfigVersion.cmake"
               "KF5DBusAddonsMacros.cmake"
               "KF5dbus.service.in"
        DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
        COMPONENT Devel )

install(EXPORT KF5DBusAddonsTargets  DESTINATION "${CMAKECONFIG_INSTALL_DIR}"  FILE KF5DBusAddonsTargets.cmake  NAMESPACE KF5:: )

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/kdbusaddons_version.h  DESTINATION ${KDE_INSTALL_INCLUDEDIR_KF5}  COMPONENT Devel )

# contains list of debug categories, for kdebugsettings
install(FILES kdbusaddons.categories DESTINATION ${KDE_INSTALL_CONFDIR})

feature_summary(WHAT ALL FATAL_ON_MISSING_REQUIRED_PACKAGES)
