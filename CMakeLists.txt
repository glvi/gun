# -*- mode: cmake; coding: utf-8; indent-tabs-mode: nil; -*-
project(gun
  VERSION 1.0.0
  DESCRIPTION "HTTP/1.1, HTTP/2 and Websocket client for Erlang/OTP"
  LANGUAGES NONE)
set(gun_source_dir
  ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(GUN_MODULES
  gun
  gun_app
  gun_content_handler
  gun_data_h
  gun_http
  gun_http2
  gun_sse_h
  gun_sup
  gun_tcp
  gun_tls
  gun_ws
  gun_ws_h)
foreach(module IN LISTS GUN_MODULES)
  list(APPEND GUN_SOURCES    ${gun_source_dir}/${module}.erl)
  list(APPEND GUN_OBJECTS    ${CMAKE_CURRENT_BINARY_DIR}/ebin/${module}.beam)
  list(APPEND GUN_MODULES_Q '${module}')
endforeach()
file(MAKE_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include)
file(RELATIVE_PATH gun_include_dir ${gun_source_dir} ${CMAKE_CURRENT_SOURCE_DIR}/include)
file(RELATIVE_PATH gun_binary_dir  ${gun_source_dir} ${CMAKE_CURRENT_BINARY_DIR}/ebin)
string(REPLACE ";" "," gun_modules "${GUN_MODULES_Q}")
configure_file(Emakefile.in ${gun_source_dir}/Emakefile)
configure_file(${gun_source_dir}/gun.app.src ebin/gun.app)
add_custom_target(Gun
  BYPRODUCTS ${GUN_OBJECTS}
  DEPENDS    ${GUN_SOURCES}
  COMMAND    ${CMAKE_COMMAND} -E make_directory ${gun_binary_dir}
  COMMAND    ${CMAKE_COMMAND} -E copy_directory ${gun_include_dir} ${CMAKE_CURRENT_BINARY_DIR}
  COMMAND    ${ERLANG_PROGRAM} -pa ${gun_binary_dir} -sasl errlog_type error -make
  WORKING_DIRECTORY ${gun_source_dir}
  VERBATIM
  COMMAND_EXPAND_LISTS)
#add_dependencies(Gun ErlangOTP)
set(URBANATM_GUN_OBJECTS ${GUN_OBJECTS} CACHE INTERNAL "UrbanATM Erlang Gun objects" FORCE)
add_custom_target(DialyzeGun
  COMMAND ${DIALYZER_PROGRAM}
          --fullpath
          --plt ${DIALYZER_PLT}
          -I ${CMAKE_CURRENT_SOURCE_DIR}/include
          -pa ${CMAKE_CURRENT_BINARY_DIR}/ebin
          --src ${CMAKE_CURRENT_SOURCE_DIR}/src
  DEPENDS ${DIALYZER_PLT}
  )
add_dependencies(DialyzeGun
  DialyzerPlt
  Gun)
