
if (APPLE)
    set(OS_BUNDLE MACOSX_BUNDLE)
elseif (WIN32)
    set(OS_BUNDLE WIN32)
endif()

set(MAIN_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/main.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/messagereceiver.h
    ${CMAKE_CURRENT_SOURCE_DIR}/klogg.qrc
    ${ICON_FILE}
)

add_executable(klogg ${OS_BUNDLE} ${MAIN_SOURCES})
add_executable(klogg_portable ${OS_BUNDLE} ${MAIN_SOURCES})

set_target_properties(klogg PROPERTIES AUTORCC ON)
set_target_properties(klogg_portable PROPERTIES AUTORCC ON)

set_property(TARGET klogg PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
set_property(TARGET klogg_portable PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)

set(MAIN_LIBS ui cli11 singleapp)
if (NOT APPLE)
    if (TBB_BUILD_TBBMALLOC)
        LIST(APPEND MAIN_LIBS klogg_tbbmalloc_proxy)
        target_compile_definitions(klogg PUBLIC -DKLOGG_USE_TBB_MALLOC)
        target_compile_definitions(klogg_portable PUBLIC -DKLOGG_USE_TBB_MALLOC)
    endif()
endif()

target_link_libraries(klogg PUBLIC ${MAIN_LIBS})
target_link_libraries(klogg_portable PUBLIC ${MAIN_LIBS})

target_compile_definitions(klogg_portable PUBLIC -DKLOGG_PORTABLE)

include(win32_rc)

if (WIN32)

    generate_product_version(
        ProductVersionResourceFiles
        NAME "${PROJECT_DESCRIPTION}"
        ORIGINAL_FILENAME ${PROJECT_NAME}
        ICON "${ICON_FILE}"
        VERSION_MAJOR ${PRODUC_VERSION_MAJOR}
        VERSION_MINOR ${PRODUC_VERSION_MINOR}
        VERSION_PATCH ${PRODUC_VERSION_PATCH}
        VERSION_REVISION ${PRODUC_VERSION_TWEAK}
        COMPANY_NAME ${COMPANY}
        COMPANY_COPYRIGHT ${COPYRIGHT}
    )

    target_sources(klogg PRIVATE ${ProductVersionResourceFiles})
    target_sources(klogg_portable PRIVATE ${ProductVersionResourceFiles})

elseif (APPLE)
    set_source_files_properties(${ICON_FILE} PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
    set_source_files_properties(${NOTICE_FILE} PROPERTIES MACOSX_PACKAGE_LOCATION SharedSupport)
    set_source_files_properties(${COPYING_FILE} PROPERTIES MACOSX_PACKAGE_LOCATION SharedSupport)

    set_source_files_properties(${README_FILE} PROPERTIES MACOSX_PACKAGE_LOCATION SharedSupport)

    set(MACOSX_BUNDLE_BUNDLE_DISPLAY_NAME ${PROJECT_DESCRIPTION})
    set(MACOSX_BUNDLE_BUNDLE_NAME ${PROJECT_NAME})
    set(MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION})
    set(MACOSX_BUNDLE_LONG_VERSION_STRING ${PROJECT_VERSION})
    set(MACOSX_BUNDLE_SHORT_VERSION_STRING "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")
    set(MACOSX_BUNDLE_COPYRIGHT ${COPYRIGHT})
    set(MACOSX_BUNDLE_GUI_IDENTIFIER ${IDENTIFIER})
    set(MACOSX_BUNDLE_ICON_FILE ${ICON_NAME})

endif()

install(TARGETS
    klogg
    DESTINATION bin
)
