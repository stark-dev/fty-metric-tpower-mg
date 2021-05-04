cmake_minimum_required(VERSION 3.13)
cmake_policy(VERSION 3.13)

project(fty-metric-tpower
    VERSION 1.0.0
    DESCRIPTION "42ity component for power metrics computation"
)

##############################################################################################################
find_package(fty-cmake PATHS ${CMAKE_BINARY_DIR}/fty-cmake)
##############################################################################################################

etn_target(exe ${PROJECT_NAME}
    SOURCES
        src/metricinfo.cc
        src/metriclist.cc
        src/tp_unit.cc
        src/tpowerconfiguration.cc
        src/watchdog.cc
        src/calc_power.cc        
        src/fty_metric_tpower.cc
        src/fty_metric_tpower_server.cc
    INCLUDE_DIRS
        include
    USES_PRIVATE        
        czmq
        mlm
        cxxtools
        fty_common
        fty_common_logging
        fty_proto
        fty_common_mlm
        fty_common_db
        fty_shm
        tntdb                
)

#install resources files
set(AGENT_SETTINGS_DIR "${CMAKE_INSTALL_FULL_LOCALSTATEDIR}/lib/fty/${PROJECT_NAME}")
set(AGENT_CONF_FILE "${CMAKE_INSTALL_FULL_SYSCONFDIR}/${PROJECT_NAME}/fty-metric-store.cfg")
set(AGENT_USER "bios")

configure_file("${PROJECT_SOURCE_DIR}/resources/${PROJECT_NAME}.cfg.in" "${PROJECT_BINARY_DIR}/resources/${PROJECT_NAME}.cfg" @ONLY)
install(FILES "${PROJECT_BINARY_DIR}/resources/${PROJECT_NAME}.cfg" DESTINATION ${CMAKE_INSTALL_FULL_SYSCONFDIR}/${PROJECT_NAME})

configure_file("${PROJECT_SOURCE_DIR}/resources/${PROJECT_NAME}.service.in" "${PROJECT_BINARY_DIR}/resources/${PROJECT_NAME}.service" @ONLY)
install(FILES "${PROJECT_BINARY_DIR}/resources/${PROJECT_NAME}.service" DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/systemd/system/)

if (BUILD_TESTING)

    etn_test(${PROJECT_NAME}-test   
        SOURCES
            tests/main.cc
            src/metricinfo.cc
            src/metriclist.cc
            src/tp_unit.cc
            src/tpowerconfiguration.cc
            src/watchdog.cc
            src/calc_power.cc                    
            src/fty_metric_tpower_server.cc
            src/fty_metric_tpower_private_selftest.cc
        INCLUDE_DIRS
            include
        PREPROCESSOR -DCATCH_CONFIG_FAST_COMPILE
        USES
            czmq
            mlm
            cxxtools
            fty_common
            fty_common_logging
            fty_proto
            fty_common_mlm
            fty_common_db
            fty_shm
            tntdb
            Catch2::Catch2
            stdc++fs
            log4cplus
    )

    #copy selftest-ro for test
    file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/tests/selftest-ro DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
    file(MAKE_DIRECTORY  ${CMAKE_CURRENT_BINARY_DIR}/selftest-rw)

    #enable coverage
    etn_coverage(${PROJECT_NAME}-test)

    #copy selftest-ro for test coverage
    file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/tests/selftest-ro DESTINATION ${CMAKE_BINARY_DIR})
    file(MAKE_DIRECTORY  ${CMAKE_BINARY_DIR}/selftest-rw)

endif()