
###
###
###  ######################################################################
###  #######                                                      #########
###  ####      SSS   PPPP   I  N    N  DDDD   U   U  M   M  PPPP       ####
###  #        S      P   P  I  NN   N  D   D  U   U  MM MM  P   P         #
###  #         SSS   PPPP   I  N NN N  D   D  U   U  M M M  PPPP          #
###  #            S  P      I  N   NN  D   D  U   U  M   M  P             #
###  ####      SSS   P      I  N    N  DDDD    UUU   M   M  P          ####
###  #######                                                      #########
###  ######################################################################
###
###  SPINDUMP (C) 2018-2019 BY ERICSSON RESEARCH
###  AUTHOR: JARI ARKKO
###
###

#
# Libraries used
#

# for libpcap installed on on MacOS via homebrew
list(APPEND CMAKE_PREFIX_PATH "/usr/local/opt/libpcap")

find_package(Curses)
find_package(Threads)
#find_package(Microhttpd)

find_path(PCAP_INCLUDE_DIR NAMES pcap/pcap.h pcap.h)
find_library(PCAP_LIBRARY NAMES pcap)
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(PCAP DEFAULT_MSG PCAP_INCLUDE_DIR PCAP_LIBRARY)

find_path(CURL_INCLUDE_DIR NAMES curl/curl.h curl.h)
find_library(CURL_LIBRARY NAMES curl)
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(CURL DEFAULT_MSG CURL_INCLUDE_DIR CURL_LIBRARY)

find_path(MICROHTTPD_INCLUDE_DIR NAMES microhttpd.h)
find_library(MICROHTTPD_LIBRARY NAMES microhttpd)
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(MICROHTTPD DEFAULT_MSG MICROHTTPD_INCLUDE_DIR MICROHTTPD_LIBRARY)

#
# Main spindump library that we create
#

add_library(spindumplib
  spindump_analyze.c
  spindump_analyze_aggregate.c
  spindump_analyze_coap.c
  spindump_analyze_dns.c
  spindump_analyze_icmp.c
  spindump_analyze_quic.c
  spindump_analyze_quic_parser.c
  spindump_analyze_tcp.c
  spindump_analyze_tls_parser.c
  spindump_analyze_udp.c
  spindump_capture.c
  spindump_connections.c
  spindump_connections_new.c
  spindump_connections_print.c
  spindump_connections_search.c
  spindump_connections_set.c
  spindump_connections_set_iterator.c
  spindump_eventformatter.c 
  spindump_eventformatter_text.c 
  spindump_eventformatter_json.c
  spindump_event.c
  spindump_event_parser_json.c
  spindump_event_parser_text.c
  spindump_mid.c
  spindump_packet.c
  spindump_protocols.c
  spindump_remote_client.c
  spindump_remote_server.c
  spindump_report.c
  spindump_reversedns.c
  spindump_reversedns_full.c
  spindump_rtt.c
  spindump_seq.c
  spindump_spin.c
  spindump_stats.c
  spindump_table.c
  spindump_util.c
)

target_include_directories(spindumplib
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)

target_link_libraries(spindumplib
  PRIVATE
    ${MICROHTTPD_LIBRARY}
    ${CURL_LIBRARY}
    ${PCAP_LIBRARY}
    ${CURSES_LIBRARIES}
    ${CMAKE_THREAD_LIBS_INIT}
    m
)

#
# Main program of the spindump tool
#

add_executable(spindump spindump_main.c)
target_link_libraries(spindump spindumplib)

#
# Testing
#

add_custom_command(
  OUTPUT spindump_test.out
  COMMAND spindump_test
  ARGS  > spindump_test.out
  DEPENDS spindump_test
)

set_property(SOURCE spindump_main.c APPEND PROPERTY OBJECT_DEPENDS src/spindump_test.out)

add_executable(spindump_test spindump_test.c)
target_link_libraries(spindump_test spindumplib)

include( CTest )

add_test( spindump_test spindump_test )

#
# Tar file
#

add_custom_command(OUTPUT ../spindump.tar.gz
                   COMMAND rm ARGS -f spindump.tar.gz
                   COMMAND tar
		   ARGS czf spindump.tar.gz CMakeLists.txt src/CMakeLists.txt README.md .gitignore LICENSE.txt images/*.png images/*.jpg src/*.h src/*.c src/*.sh
		   WORKING_DIRECTORY ..
)

set_property(SOURCE spindump_main.c APPEND PROPERTY OBJECT_DEPENDS spindump.tar.gz)

#
# Installations
#


install(SCRIPT spindump_install.sh)

add_custom_target(uninstall
                  COMMAND sh spindump_uninstall.sh
)
		  
#
# Word counts
#

add_custom_target(wc
                  COMMAND sh spindump_wc.sh
)
