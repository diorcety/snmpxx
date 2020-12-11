cmake_minimum_required(VERSION 3.6)

project(snmpxx)

set (CMAKE_CXX_STANDARD 11)

set(SNMP_PP_MAJOR_VERSION 3)
set(SNMP_PP_MINOR_VERSION 4)
set(SNMP_PP_MICRO_VERSION 2)
set(VERSION ${SNMP_PP_MAJOR_VERSION}.${SNMP_PP_MINOR_VERSION}.${SNMP_PP_MICRO_VERSION})

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

option(ENABLE_SNMPV3 "Enable SNMPV3" ON)
option(ENABLE_IPV6 "Enable IPV6" ON)
option(ENABLE_NAMESPACE "Enable namespace" ON)
option(ENABLE_LOGGING "Enable logging" ON)
option(ENABLE_THREADS "Enable thread support" ON)
option(ENABLE_MACADDRESS "Enable support of MacAddress" OFF)
option(ENABLE_IPXADDRESS "Enable support of IpxAddress" OFF)

find_package(OpenSSL)
include_directories(${OPENSSL_INCLUDE_DIR})
find_package(Threads)

include(CheckIncludeFiles)
include(CheckFunctionExists)
include(CheckTypeSize)
include(CheckStructHasMember)
include(CheckLibraryExists)
include(CheckSymbolExists)

#add_definitions(-DHAVE_CONFIG_H)
add_definitions(-DSNMP_PP_EXPORTS)

macro(_check_include_files FILES VAR)
  check_include_files("${FILES}" ${VAR})
  if (NOT ${VAR})
    set(${VAR} 0)
  endif()
endmacro()

macro(_check_include_files_cxx FILES VAR)
  check_include_files("${FILES}" ${VAR} LANGUAGE CXX)
  if (NOT ${VAR})
    set(${VAR} 0)
  endif()
endmacro()

macro(_check_include_file FILE VAR)
  check_include_file(${FILE} ${VAR})
  if (NOT ${VAR})
    set(${VAR} 0)
  endif()
endmacro()

macro(_check_function_exists FUNCTION VAR)
  check_function_exists(${FUNCTION} ${VAR})
  if (NOT ${VAR})
    set(${VAR} 0)
  endif()
endmacro()

macro(_check_type_size TYPE VAR)
  check_type_size(${TYPE} ${VAR})
  if (NOT ${VAR})
    set(${VAR} 0)
  endif()
endmacro()

macro(_check_struct_has_member STRUCT MEMBER HEADERS VAR)
  check_struct_has_member("${STRUCT}" ${MEMBER} "${HEADERS}" ${VAR})
  if (NOT ${VAR})
    set(${VAR} 0)
  endif()
endmacro()

_check_include_files_cxx("cctype;cerrno;climits;csignal;cstddef;cstdio;cstdlib;cstring;ctime" CNF_STDCXX_98_HEADERS)
_check_include_files("stdlib.h;stdarg.h;string.h;float.h" CNF_STDC_HEADERS)
_check_include_file(float.h CNF_HAVE_FLOAT_H)
_check_include_file(sys/types.h CNF_HAVE_SYS_TYPES_H)
_check_include_file(sys/stat.h CNF_HAVE_SYS_STAT_H)
_check_include_file(stdlib.h CNF_HAVE_STDLIB_H)
_check_include_file(string.h CNF_HAVE_STRING_H)
_check_include_file(memory.h CNF_HAVE_MEMORY_H)
_check_include_file(strings.h CNF_HAVE_STRINGS_H)
_check_include_file(inttypes.h CNF_HAVE_INTTYPES_H)
_check_include_file(ctype.h CNF_HAVE_CTYPE_H)
_check_include_file(signal.h CNF_HAVE_SIGNAL_H)
_check_include_file(dlfcn.h CNF_HAVE_DLFCN_H)
_check_include_file(errno.h CNF_HAVE_ERRNO_H)
_check_include_file(time.h CNF_HAVE_TIME_H)
_check_include_file(unistd.h CNF_HAVE_UNISTD_H)
_check_include_file(sys/unistd.h CNF_HAVE_SYS_UNISTD_H)
_check_include_file(stdint.h CNF_HAVE_STDINT_H)
_check_include_file(sys/time.h CNF_HAVE_SYS_TIME_H)
_check_include_file(sys/param.h CNF_HAVE_SYS_PARAM_H)
_check_include_file(sys/timeb.h CNF_HAVE_SYS_TIMEB_H)
_check_include_file(winsock2.h CNF_HAVE_WINSOCK2_H)
_check_include_file(ws2tcpip.h CNF_HAVE_WS2TCPIP_H)
_check_include_file(wspiapi.h CNF_HAVE_WSPIAPI_H)
_check_include_file(winsock.h CNF_HAVE_WINSOCK_H)
_check_include_file(netdb.h CNF_HAVE_NETDB_H)
_check_include_file(sys/socket.h CNF_HAVE_SYS_SOCKET_H)
_check_include_file(arpa/inet.h CNF_HAVE_ARPA_INET_H)
_check_include_file(netinet/in.h CNF_HAVE_NETINET_IN_H)
_check_include_file(netinet/in6.h CNF_HAVE_NETINET_IN6_H)
_check_include_file(poll.h CNF_HAVE_POLL_H)
_check_include_file(sys/select.h CNF_HAVE_SYS_SELECT_H)
_check_include_file(io.h CNF_HAVE_IO_H)
_check_include_file(process.h CNF_HAVE_PROCESS_H)

check_library_exists(rt clock_gettime "" HAVE_CLOCK_GETTIME)
if (NOT HAVE_CLOCK_GETTIME)
   set(CMAKE_EXTRA_INCLUDE_FILES_OLD ${CMAKE_EXTRA_INCLUDE_FILES})
   set(CMAKE_EXTRA_INCLUDE_FILES time.h)
   _check_function_exists(clock_gettime HAVE_CLOCK_GETTIME)
   set(CMAKE_EXTRA_INCLUDE_FILES ${CMAKE_EXTRA_INCLUDE_FILES_OLD})
endif()

check_symbol_exists(getaddrinfo netdb.h CNF_HAVE_DECL_GETADDRINFO)

_check_function_exists(strcasecmp CNF_HAVE_STRCASECMP)
_check_function_exists(stricmp CNF_HAVE_STRICMP)
_check_function_exists(getpid CNF_HAVE_GETPID)
_check_function_exists(_getpid CNF_HAVE__GETPID)

check_library_exists("wsock32" gethostbyname "" HAVE_LIBWSOCK32)

# Check for sockaddr structure sizes.
set(SOCKADDR_HEADERS)
if (WIN32)
    set(CMAKE_REQUIRED_DEFINITIONS "-DWIN32_LEAN_AND_MEAN")
    if (_MSC_VER LESS 1300)
        set(SOCKADDR_HEADERS winsock.h)
    else()
        set(SOCKADDR_HEADERS winsock2.h ws2tcpip.h)
    endif()
else()
    if (CNF_HAVE_NETINET_IN_H)
        set(SOCKADDR_HEADERS ${SOCKADDR_HEADERS} netinet/in.h)
    endif()

    if (CNF_HAVE_NETINET_IN6_H)
        set(SOCKADDR_HEADERS ${SOCKADDR_HEADERS} netinet/in6.h)
    endif()

    if (CNF_HAVE_SYS_SOCKET_H)
        set(SOCKADDR_HEADERS ${SOCKADDR_HEADERS} sys/socket.h)
    endif()

    if (CNF_HAVE_NETDB_H)
        set(SOCKADDR_HEADERS ${SOCKADDR_HEADERS} netdb.h)
    endif()
endif()

set(CMAKE_EXTRA_INCLUDE_FILES_OLD ${CMAKE_EXTRA_INCLUDE_FILES})
list(APPEND CMAKE_EXTRA_INCLUDE_FILES ${SOCKADDR_HEADERS})
_check_type_size("struct sockaddr_storage" CNF_HAVE_STRUCT_SOCKADDR_STORAGE)
set(CNF_HAVE_STRUCT_SOCKADDR_STORAGE_SS_FAMILY 0)
set(CNF_HAVE_STRUCT_SOCKADDR_STORAGE___SS_FAMILY 0)
if(CNF_HAVE_STRUCT_SOCKADDR_STORAGE)
    _check_struct_has_member("struct sockaddr_storage" ss_family "${SOCKADDR_HEADERS}" CNF_HAVE_STRUCT_SOCKADDR_STORAGE_SS_FAMILY)

    _check_struct_has_member("struct sockaddr_storage" __ss_family "${SOCKADDR_HEADERS}" CNF_HAVE_STRUCT_SOCKADDR_STORAGE___SS_FAMILY)
endif()
set(CMAKE_EXTRA_INCLUDE_FILES ${CMAKE_EXTRA_INCLUDE_FILES_OLD})


CHECK_TYPE_SIZE(int8_t INT8_T)
CHECK_TYPE_SIZE(int16_t INT16_T)
CHECK_TYPE_SIZE(int32_t INT32_T)
CHECK_TYPE_SIZE(int64_t INT64_T)
CHECK_TYPE_SIZE(uint8_t UINT8_T)
CHECK_TYPE_SIZE(uint16_t UINT16_T)
CHECK_TYPE_SIZE(uint32_t UINT32_T)
CHECK_TYPE_SIZE(uint64_t UINT64_T)

CHECK_TYPE_SIZE(gid_t       GID_T)
IF(NOT HAVE_GID_T)
  IF(WIN32)
    SET(gid_t "short")
  ELSE(WIN32)
    SET(gid_t "unsigned int")
  ENDIF(WIN32)
ENDIF(NOT HAVE_GID_T)

CHECK_TYPE_SIZE(off_t       OFF_T)
IF(NOT HAVE_OFF_T)
  SET(off_t "__int64")
ENDIF(NOT HAVE_OFF_T)

CHECK_TYPE_SIZE(size_t      SIZE_T)
IF(NOT HAVE_SIZE_T)
  IF("${CMAKE_SIZEOF_VOID_P}" EQUAL 8)
    SET(size_t "uint64_t")
  ELSE("${CMAKE_SIZEOF_VOID_P}" EQUAL 8)
    SET(size_t   "uint32_t")
  ENDIF("${CMAKE_SIZEOF_VOID_P}" EQUAL 8)
ENDIF(NOT HAVE_SIZE_T) 

CHECK_TYPE_SIZE(ssize_t     SSIZE_T)
IF(NOT HAVE_SSIZE_T)
  IF("${CMAKE_SIZEOF_VOID_P}" EQUAL 8)
    SET(ssize_t "int64_t")
  ELSE("${CMAKE_SIZEOF_VOID_P}" EQUAL 8)
    SET(ssize_t "long")
  ENDIF("${CMAKE_SIZEOF_VOID_P}" EQUAL 8)
ENDIF(NOT HAVE_SSIZE_T)

CHECK_TYPE_SIZE(uid_t       UID_T)
IF(NOT HAVE_UID_T)
  IF(WIN32)
    SET(uid_t "short")
  ELSE(WIN32)
    SET(uid_t "unsigned int")
  ENDIF(WIN32)
ENDIF(NOT HAVE_UID_T)

CHECK_TYPE_SIZE(pid_t       PID_T)
IF(NOT HAVE_PID_T)
  IF(WIN32)
    SET(pid_t "int")
  ELSE(WIN32)
    MESSAGE(FATAL_ERROR "pid_t doesn't exist on this platform?")
  ENDIF(WIN32)
ENDIF(NOT HAVE_PID_T)

set(WITH_LOGGING 0)
set(WITH_LOG_PROFILES 0)
set(WITH_MODULES )

set(WITH_PTHREAD 0)
set(WITH_THREADS 0)
if(ENABLE_THREADS)
  set(WITH_THREADS 1)
  if(CMAKE_USE_PTHREADS_INIT)
    set(WITH_PTHREAD 1)
  endif(CMAKE_USE_PTHREADS_INIT)
endif(ENABLE_THREADS)

set(WITH_MACADDRESS 0)
if(ENABLE_MACADDRESS)
  set(WITH_MACADDRESS 1)
endif(ENABLE_MACADDRESS)

set(WITH_IPXADDRESS 0)
if(ENABLE_IPXADDRESS)
  set(WITH_IPXADDRESS 1)
endif(ENABLE_IPXADDRESS)

set(WITH_NAMESPACE 0)
if(ENABLE_NAMESPACE)
  set(WITH_NAMESPACE 1)
endif(ENABLE_NAMESPACE)

set(WITH_LIBTOMCRYPT 0)
set(WITH_LIBDES 0)
set(WITH_LIBSSL 0)
if(OPENSSL_FOUND)
  set(WITH_LIBSSL 1)
endif(OPENSSL_FOUND)

set(_SNMPv3 0)
if(ENABLE_SNMPV3)
  if(NOT OPENSSL_FOUND)
    message( FATAL_ERROR "cryptographic library required for SNMPv3" )
  endif()
  set(_SNMPv3 1)
endif(ENABLE_SNMPV3)

set(WITH_SNMP_PP_IPv6 0)
if(ENABLE_IPV6)
  set(WITH_SNMP_PP_IPv6 1)
endif(ENABLE_IPV6)

set(LIBRARIES )
set(LIBRARIES ${LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
if(OPENSSL_FOUND)
  set(LIBRARIES ${LIBRARIES} ${OPENSSL_LIBRARIES})
endif()
if(WIN32)
  set(LIBRARIES ${LIBRARIES} ws2_32)
endif()

function(generate_pkgconfig)
  set(PACKAGE_NAME "${PROJECT_NAME}")
  set(PACKAGE_VERSION "${LIBFFI_VERSION}")
  set(prefix "${CMAKE_INSTALL_PREFIX}")
  set(exec_prefix "\${prefix}")
  set(libdir "\${exec_prefix}/lib")
  set(toolexeclibdir "\${exec_prefix}/lib/../lib")
  set(includedir "\${prefix}/include")
  set(LIBS "")
  foreach(f ${LIBRARIES})
    string(FIND "${f}" "-l" out)
    if ("${out}" EQUAL 0)
      set(LIBS "${LIBS} ${f}")
    else()
      set(LIBS "${LIBS} -l${f}")
    endif()
  endforeach(f)

  configure_file("${CMAKE_CURRENT_SOURCE_DIR}/snmp++-3.pc.in" "${CMAKE_CURRENT_BINARY_DIR}/snmp++-3.pc" @ONLY)
endfunction()

generate_pkgconfig()

set(INCLUDE_FILES
    include/snmp_pp/vb.h
    include/snmp_pp/gauge.h
    include/snmp_pp/auth_priv.h
    include/snmp_pp/md5.h
    include/snmp_pp/target.h
    include/snmp_pp/IPv6Utility.h
    include/snmp_pp/counter.h
    include/snmp_pp/ctr64.h
    include/snmp_pp/msgqueue.h
    include/snmp_pp/uxsnmp.h
    include/snmp_pp/usm_v3.h
    include/snmp_pp/oid.h
    include/snmp_pp/idea.h
    include/snmp_pp/snmp_pp.h
    include/snmp_pp/integer.h
    include/snmp_pp/sha.h
    include/snmp_pp/address.h
    include/snmp_pp/snmperrs.h
    include/snmp_pp/log.h
    include/snmp_pp/mp_v3.h
    include/snmp_pp/eventlist.h
    include/snmp_pp/smi.h
    include/snmp_pp/asn1.h
    include/snmp_pp/snmpmsg.h
    include/snmp_pp/v3.h
    include/snmp_pp/smival.h
    include/snmp_pp/collect.h
    include/snmp_pp/octet.h
    include/snmp_pp/eventlistholder.h
    include/snmp_pp/notifyqueue.h
    include/snmp_pp/timetick.h
    include/snmp_pp/reentrant.h
    include/snmp_pp/oid_def.h
    include/snmp_pp/pdu.h
    include/snmp_pp/msec.h
)

#configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.cmake.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/config.h)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/include/snmp_pp/config_snmp_pp.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/snmp_pp/config_snmp_pp.h @ONLY)
set(INCLUDE_FILES ${INCLUDE_FILES} ${CMAKE_CURRENT_BINARY_DIR}/include/snmp_pp/config_snmp_pp.h)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/libsnmp.h.in ${CMAKE_CURRENT_BINARY_DIR}/include/libsnmp.h @ONLY)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories(${CMAKE_CURRENT_BINARY_DIR}/include)

set(SOURCE_FILES
    src/address.cpp
    src/asn1.cpp
    src/auth_priv.cpp
    src/counter.cpp
    src/ctr64.cpp
    src/eventlist.cpp
    src/eventlistholder.cpp
    src/gauge.cpp
    src/idea.cpp
    src/integer.cpp
    src/IPv6Utility.cpp
    src/log.cpp
    src/md5c.cpp
    src/mp_v3.cpp
    src/msec.cpp
    src/msgqueue.cpp
    src/notifyqueue.cpp
    src/octet.cpp
    src/oid.cpp
    src/pdu.cpp
    src/reentrant.cpp
    src/sha.cpp
    src/snmpmsg.cpp
    src/target.cpp
    src/timetick.cpp
    src/usm_v3.cpp
    src/uxsnmp.cpp
    src/v3.cpp
    src/vb.cpp
)

add_library(libsnmp ${SOURCE_FILES} ${INCLUDE_FILES})
target_link_libraries(libsnmp ${LIBRARIES})
set_target_properties(libsnmp PROPERTIES OUTPUT_NAME "snmp++")
set_target_properties(libsnmp PROPERTIES PUBLIC_HEADER "${INCLUDE_FILES}")

install(TARGETS libsnmp
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/snmp_pp")

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/snmp++-3.pc" DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/include/libsnmp.h" DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

