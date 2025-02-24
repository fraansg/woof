include(WoofSettings)

set(WOOF_SOURCES
    am_map.c               am_map.h
                           beta.h
    d_deh.c                d_deh.h
                           d_englsh.h
                           d_event.h
                           d_french.h
    d_items.c              d_items.h
    d_iwad.c               d_iwad.h
    d_loop.c               d_loop.h
    d_main.c               d_main.h
    d_net.c
                           d_player.h
    d_quit.c               d_quit.h
                           d_textur.h
                           d_think.h
                           d_ticcmd.h
                           dogs.h
                           doomdata.h
    doomdef.c              doomdef.h
    doomstat.c             doomstat.h
                           doomtype.h
    dsdhacked.c            dsdhacked.h
    dstrings.c             dstrings.h
    f_finale.c             f_finale.h
    f_wipe.c               f_wipe.h
    g_game.c               g_game.h
    hu_lib.c               hu_lib.h
    hu_stuff.c             hu_stuff.h
    i_endoom.c             i_endoom.h
    i_flmusic.c
    i_glob.c               i_glob.h
    i_main.c
    i_oplmusic.c
    i_sdlmusic.c
    i_sndfile.c            i_sndfile.h
    i_sound.c              i_sound.h
    i_system.c             i_system.h
    i_timer.c              i_timer.h
    i_video.c              i_video.h
    info.c                 info.h
    m_argv.c               m_argv.h
    m_bbox.c               m_bbox.h
    m_cheat.c              m_cheat.h
                           m_fixed.h
    m_input.c              m_input.h
    m_io.c                 m_io.h
    m_menu.c               m_menu.h
    m_misc.c               m_misc.h
    m_misc2.c              m_misc2.h
    m_random.c             m_random.h
    m_snapshot.c           m_snapshot.h
                           m_swap.h
    memio.c                memio.h
    midifallback.c         midifallback.h
    midifile.c             midifile.h
    mus2mid.c              mus2mid.h
    net_client.c           net_client.h
    net_common.c           net_common.h
    net_dedicated.c        net_dedicated.h
                           net_defs.h
    net_gui.c              net_gui.h
    net_io.c               net_io.h
    net_loop.c             net_loop.h
    net_packet.c           net_packet.h
    net_petname.c          net_petname.h
    net_query.c            net_query.h
    net_sdl.c              net_sdl.h
    net_server.c           net_server.h
    net_structrw.c         net_structrw.h
                           p_action.h
    p_ceilng.c
    p_doors.c
    p_enemy.c              p_enemy.h
    p_extnodes.c           p_extnodes.h
    p_floor.c
    p_genlin.c
    p_inter.c              p_inter.h
    p_lights.c
    p_map.c                p_map.h
    p_maputl.c             p_maputl.h
    p_mobj.c               p_mobj.h
    p_plats.c
    p_pspr.c               p_pspr.h
    p_saveg.c              p_saveg.h
    p_setup.c              p_setup.h
    p_sight.c
    p_spec.c               p_spec.h
    p_switch.c
    p_telept.c
    p_tick.c               p_tick.h
    p_user.c               p_user.h
    r_bmaps.c              r_bmaps.h
    r_bsp.c                r_bsp.h
    r_data.c               r_data.h
                           r_defs.h
    r_draw.c               r_draw.h
    r_main.c               r_main.h
    r_plane.c              r_plane.h
    r_segs.c               r_segs.h
    r_sky.c                r_sky.h
                           r_state.h
    r_swirl.c              r_swirl.h
    r_things.c             r_things.h
    s_musinfo.c            s_musinfo.h
    s_sound.c              s_sound.h
    sounds.c               sounds.h
    st_lib.c               st_lib.h
    st_stuff.c             st_stuff.h
    statdump.c             statdump.h
    tables.c               tables.h
                           thermo.h
    u_mapinfo.c            u_mapinfo.h
    u_scanner.c            u_scanner.h
    v_video.c              v_video.h
    version.c              version.h
    w_wad.c                w_wad.h
    wi_stuff.c             wi_stuff.h
    z_zone.c               z_zone.h
    ../miniz/miniz.c       ../miniz/miniz.h)

if(WIN32)
    list(APPEND
        WOOF_SOURCES
        i_winmusic.c
        ../win32/win_version.c ../win32/win_version.h)
    if(MSVC)
        list(APPEND
            WOOF_SOURCES
            ../win32/win_opendir.c ../win32/win_opendir.h)
    endif()
endif()

# Some platforms require standard libraries to be linked against.
if(HAVE_LIBM)
    list(APPEND WOOF_LIBRARIES m)
endif()

if (WIN32)
    list(APPEND WOOF_LIBRARIES winmm)
endif()

# Standard target definition
if(WIN32)
    add_library(woof SHARED EXCLUDE_FROM_ALL ${WOOF_SOURCES})
    target_compile_definitions(woof PRIVATE WIN_LAUNCHER)

    list(APPEND WIN_LAUNCHER_LIBS "$<TARGET_NAME_IF_EXISTS:SDL2::SDL2main>"
        SDL2::SDL2 woof)

    # Stamp out and compile resource file on Windows.
    configure_file(resource.rc.in "${CMAKE_CURRENT_BINARY_DIR}/resource.rc")

    add_executable(woof-com WIN32 ../win32/win_launcher.c
        "${CMAKE_CURRENT_BINARY_DIR}/resource.rc")
    target_woof_settings(woof-com)
    target_link_libraries(woof-com PRIVATE ${WIN_LAUNCHER_LIBS})
    # MSVC tries to supply a default manifest and complains when it finds ours
    # unless we specifically tell it not to.
    if(MSVC)
        target_link_options(woof-com PRIVATE "/MANIFEST:NO" "/SUBSYSTEM:CONSOLE")
    else()
        target_link_options(woof-com PRIVATE "-mconsole")
    endif()

    add_executable(woof-exe WIN32 ../win32/win_launcher.c
        "${CMAKE_CURRENT_BINARY_DIR}/resource.rc")
    target_woof_settings(woof-exe)
    target_link_libraries(woof-exe PRIVATE ${WIN_LAUNCHER_LIBS})
    if(MSVC)
        target_link_options(woof-exe PRIVATE "/MANIFEST:NO")
    endif()
    set_target_properties(woof-exe PROPERTIES RUNTIME_OUTPUT_NAME "woof"
        SUFFIX ".exe")
else()
    list(APPEND WOOF_LIBRARIES "$<TARGET_NAME_IF_EXISTS:SDL2::SDL2main>")
    add_executable(woof ${WOOF_SOURCES})
endif()

target_woof_settings(woof)
target_include_directories(woof PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/../")

if (FluidSynth_FOUND)
    list(APPEND WOOF_LIBRARIES FluidSynth::libfluidsynth)
    target_compile_definitions(woof PRIVATE HAVE_FLUIDSYNTH)
endif()

if (SndFile_FOUND)
    list(APPEND WOOF_LIBRARIES SndFile::sndfile)
    target_compile_definitions(woof PRIVATE HAVE_SNDFILE)
endif()

target_link_libraries(woof PRIVATE ${WOOF_LIBRARIES}
    SDL2::SDL2 SDL2_mixer::SDL2_mixer SDL2_net::SDL2_net opl textscreen)

# Optional features.
#
# Our defines are not namespaced, so we pass them at compile-time instead of
# using config.h.
if("${${PROJECT_NAME}_RANGECHECK}")
    target_compile_definitions(woof PRIVATE RANGECHECK)
endif()
if("${${PROJECT_NAME}_STRICT}")
    target_compile_definitions(woof PRIVATE MBF_STRICT)
endif()

target_compile_definitions(woof PRIVATE MINIZ_NO_TIME)

set(SETUP_SOURCES
    d_iwad.c                d_iwad.h
    i_main.c
    i_system.c              i_system.h
    i_timer.c               i_timer.h
    m_argv.c                m_argv.h
    m_io.c                  m_io.h
    m_misc2.c               m_misc2.h
    net_io.c                net_io.h
    net_packet.c            net_packet.h
    net_query.c             net_query.h
    net_sdl.c               net_sdl.h
    net_structrw.c          net_structrw.h
    version.c               version.h
    z_zone.c                z_zone.h)

if(WIN32)
    # Stamp out and compile resource file on Windows.
    configure_file(setup-res.rc.in "${CMAKE_CURRENT_BINARY_DIR}/setup-res.rc")
    list(APPEND SETUP_SOURCES "${CMAKE_CURRENT_BINARY_DIR}/setup-res.rc")
endif()

add_executable(woof-setup WIN32 ${SETUP_SOURCES})
target_woof_settings(woof-setup)

target_include_directories(woof-setup PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/../")
target_link_libraries(woof-setup PRIVATE "$<TARGET_NAME_IF_EXISTS:SDL2::SDL2main>"
    SDL2::SDL2 SDL2_net::SDL2_net textscreen setup)

if(MSVC)
    target_link_options(woof-setup PRIVATE "/MANIFEST:NO")
endif()

# Install
#
# Rename woof-com executable.
if (WIN32)
    add_custom_command(TARGET woof-com POST_BUILD
        COMMAND "${CMAKE_COMMAND}" -E copy
        "$<TARGET_FILE_DIR:woof-com>/$<TARGET_FILE_NAME:woof-com>"
        "$<TARGET_FILE_DIR:woof-com>/woof.com")
endif()

# Files to package in our distribution.
if(WIN32)
    install(TARGETS woof woof-exe woof-setup RUNTIME DESTINATION .)
    install(FILES "$<TARGET_FILE_DIR:woof-com>/woof.com" DESTINATION .)
else()
    install(TARGETS woof woof-setup RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
endif()

# Copy autoload directory content to target directory.
add_custom_command(TARGET woof POST_BUILD
    COMMAND "${CMAKE_COMMAND}" -E make_directory
    "$<TARGET_FILE_DIR:woof>/autoload"
    COMMAND "${CMAKE_COMMAND}" -E copy_directory
    "${CMAKE_CURRENT_SOURCE_DIR}/../autoload"
    "$<TARGET_FILE_DIR:woof>/autoload" VERBATIM)
