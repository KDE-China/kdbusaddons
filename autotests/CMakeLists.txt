
remove_definitions(-DQT_NO_CAST_FROM_ASCII)

find_package(Qt5Test ${REQUIRED_QT_VERSION} REQUIRED NO_MODULE)

include(ECMAddTests)

ecm_add_tests(
    kdbusinterprocesslocktest.cpp
    kdbusservicetest.cpp
    LINK_LIBRARIES Qt5::Test KF5::DBusAddons
)
