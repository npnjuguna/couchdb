FILE(GLOB SRC *.erl)
SET(MR_VERSION "1.0")
SET(MR_PREFIX ${CMAKE_ERL_LIB_INSTALL_PREFIX}/mapreduce-${MR_VERSION})
SET(MR_EBIN_PREFIX ${MR_PREFIX}/ebin)
SET(MR_PRIV_PREFIX ${MR_PREFIX}/priv)


CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/test/run.tpl
               ${CMAKE_CURRENT_BINARY_DIR}/test/run)

CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/mapreduce.app.in
               ${CMAKE_CURRENT_BINARY_DIR}/mapreduce.app)



SET(COUCH_VIEW_MR_SRC mapreduce_nif.cc
                      mapreduce.cc)

INCLUDE_DIRECTORIES(BEFORE ${V8_INCLUDE_DIR}
                           ${Platform_SOURCE_DIR}/include
                           ${ERLANG_INCLUDE_PATH})

ADD_LIBRARY(mapreduce_nif MODULE ${COUCH_VIEW_MR_SRC})
SET_TARGET_PROPERTIES(mapreduce_nif PROPERTIES PREFIX "")

TARGET_LINK_LIBRARIES(mapreduce_nif platform ${COUCHBASE_UNRESOLVED}
                                    ${V8_LIBRARIES}
                                    ${COUCHBASE_MATH_LIBS})
SET_TARGET_PROPERTIES(mapreduce_nif PROPERTIES
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/priv"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/priv"
)


ERL_BUILD("mapreduce" ${SRC})

INSTALL(TARGETS mapreduce_nif
        DESTINATION ${MR_PRIV_PREFIX})
INSTALL(FILES ${outfiles}
               ${CMAKE_CURRENT_BINARY_DIR}/mapreduce.app
        DESTINATION ${MR_EBIN_PREFIX})

ADD_SUBDIRECTORY(test)
