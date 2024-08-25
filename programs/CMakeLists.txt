# Build program.
add_executable(gpmetis gpmetis.c cmdline_gpmetis.c io.c stat.c)
add_executable(ndmetis ndmetis.c cmdline_ndmetis.c io.c smbfactor.c)
add_executable(mpmetis mpmetis.c cmdline_mpmetis.c io.c stat.c)
add_executable(m2gmetis m2gmetis.c cmdline_m2gmetis.c io.c)
add_executable(graphchk graphchk.c io.c)
add_executable(cmpfillin cmpfillin.c io.c smbfactor.c)

# Link with the required libraries
foreach(prog gpmetis ndmetis mpmetis m2gmetis graphchk cmpfillin)

    # set common options and warnings
    ProjectCompilerSettings(${prog})

    target_link_libraries(${prog} metis GKlib::GKlib)

    if(UNIX)
        target_link_libraries(${prog} m)
    endif()

endforeach(prog)

# add some simple command line tools tests to verify functioning build
add_test(NAME gpmetis
    COMMAND gpmetis ${CMAKE_CURRENT_SOURCE_DIR}/../graphs/4elt.graph 4
)

add_test(NAME ndmetis
    COMMAND ndmetis ${CMAKE_CURRENT_SOURCE_DIR}/../graphs/4elt.graph
)

add_test(NAME mpmetis
    COMMAND mpmetis ${CMAKE_CURRENT_SOURCE_DIR}/../graphs/metis.mesh 4
)

add_test(NAME m2gmetis
    COMMAND m2gmetis ${CMAKE_CURRENT_SOURCE_DIR}/../graphs/metis.mesh output.graph
)

add_test(NAME graphchk 
    COMMAND graphchk ${CMAKE_CURRENT_SOURCE_DIR}/../graphs/4elt.graph
)

if(WIN32)
    get_property(test_list DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY TESTS)

    foreach(test ${test_list})
        set_tests_properties(${test} PROPERTIES
            ENVIRONMENT_MODIFICATION PATH=path_list_prepend:$<TARGET_FILE_DIR:metis>
        )
    endforeach()
endif()