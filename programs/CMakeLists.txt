# Build program.
add_executable(gpmetis gpmetis.c cmdline_gpmetis.c io.c stat.c)
add_executable(ndmetis ndmetis.c cmdline_ndmetis.c io.c smbfactor.c)
add_executable(mpmetis mpmetis.c cmdline_mpmetis.c io.c stat.c)
add_executable(m2gmetis m2gmetis.c cmdline_m2gmetis.c io.c)
add_executable(graphchk graphchk.c io.c)
add_executable(cmpfillin cmpfillin.c io.c smbfactor.c)

# Link with the required libraries
foreach(prog gpmetis ndmetis mpmetis m2gmetis graphchk cmpfillin)
    # set compiler defines
    target_compile_definitions(${prog} PRIVATE
        $<$<PLATFORM_ID:Windows>: WIN32 USE_GKREGEX >
        $<$<PLATFORM_ID:Linux>: LINUX _FILE_OFFSET_BITS=64 >
        $<$<PLATFORM_ID:CYGWIN>: CYGWIN >
        $<$<C_COMPILER_ID:MSVC>: MSC _CRT_SECURE_NO_DEPRECATE >
        $<$<AND:$<BOOL:${HAVE_THREADLOCALSTORAGE}>,$<C_COMPILER_ID:MSVC>>: "__thread=__declspec(thread)" >
        $<$<AND:$<NOT:$<BOOL:${HAVE_THREADLOCALSTORAGE}>>,$<C_COMPILER_ID:MSVC>>: __thread= >
    )

    # set warnings
    target_compile_options(${prog} PRIVATE
        # GNU options
        $<$<C_COMPILER_ID:GNU>: -Werror
            -Wall 
            -pedantic 
            -Wno-unused-function 
            -Wno-unused-but-set-variable 
            -Wno-unused-variable 
            -Wno-unknown-pragmas 
            -Wno-unused-label 
            -Wno-int-to-pointer-cast
            -Wno-pointer-to-int-cast
            -Wno-maybe-uninitialized >

        # Clang options
        $<$<C_COMPILER_ID:Clang>: -Wno-deprecated-declarations 
            -Wno-implicit-function-declaration 
            -Wno-format 
            -Wno-format-invalid-specifier 
            -Wno-format-extra-args 
            -Wno-format-invalid-specifier 
            -Wno-deprecated-non-prototype 
            -Wno-int-to-void-pointer-cast 
            -Wno-void-pointer-to-int-cast 
            -Wno-unused-variable 
            -Wno-unused-function 
            -Wno-unused-but-set-variable >

        # MSVC options
        $<$<C_COMPILER_ID:MSVC>: /wd4005 /wd4311 /wd4312 >
    )

    target_link_libraries(${prog} metis GKlib::GKlib)

    if(UNIX)
        target_link_libraries(${prog} m)
    endif()

endforeach(prog)
