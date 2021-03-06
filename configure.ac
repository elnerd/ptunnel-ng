AC_PREREQ(2.69)
AC_INIT([ptunnel-ng], [1.31], [], [], [])
AC_CONFIG_SRCDIR([src/config.h.in])
AC_CONFIG_FILES([Makefile src/Makefile])

AC_CANONICAL_BUILD
AC_CANONICAL_HOST
case x"${host}" in
	x*-*-cygwin* | x*-*-mingw32*)
		dnl Some Windows includes required by third-party modules.
		use_msw=yes
		PROGRAM_EXT=".exe"
	;;
	*-androideabi)
		use_android=yes
	;;
esac

AM_SILENT_RULES([yes])
AM_INIT_AUTOMAKE
AC_PROG_INSTALL
AC_PROG_CC
AC_PROG_CC_STDC
AC_USE_SYSTEM_EXTENSIONS
AC_C_CONST
AC_C_INLINE
AC_TYPE_SIZE_T
AC_CHECK_HEADER_STDBOOL
AC_FUNC_VPRINTF
AC_FUNC_MEMCMP

dnl Check for -std=gnu99
if test x"${ac_cv_prog_cc_stdc}" = x"no"; then
	AC_MSG_ERROR([Your compiler does not support \`-std=gnu99\`. This is fatal.])
fi

dnl Check for std includes.
AC_CHECK_HEADERS([stdarg.h stdio.h unistd.h stdlib.h string.h stdint.h time.h signal.h assert.h],,
    [AC_MSG_ERROR([Missing essential std headers.])])
if test x"${use_msw}" != x"yes"; then
	AC_CHECK_HEADERS([sys/unistd.h],,)
	AC_CHECK_HEADERS([sys/types.h sys/socket.h netinet/in.h arpa/inet.h netdb.h pthread.h errno.h net/ethernet.h syslog.h pwd.h grp.h],,
	    [AC_MSG_ERROR([Missing essential non-Windows std headers.])])
	AC_SEARCH_LIBS([pthread_create], [pthread],,
	    [AC_MSG_ERROR([Missing pthread library.])],)
	if test x"${use_android}" != x"yes"; then
		AC_CHECK_FUNCS([signal],,[AC_MSG_ERROR([Missing essential non-Android std functions.])])
	fi
	AC_CHECK_FUNCS([pthread_mutex_init pthread_mutex_lock pthread_mutex_unlock syslog getaddrinfo freeaddrinfo gai_strerror],,
	    [AC_MSG_ERROR([Missing essential Linux std functions.])])
else
	AC_CHECK_TOOL([DLLTOOL], [dlltool], [:])
	if test x"${DLLTOOL}" = x":"; then
		AC_MSG_ERROR([Missing dlltool which is required to build the import library!])
	fi
	AC_CHECK_HEADERS([winsock2.h windows.h ws2tcpip.h],,
	    [AC_MSG_ERROR([Missing essential Windows std headers.])])
	AC_CHECK_LIB([ws2_32],[main])
fi

AC_MSG_CHECKING([for GNU getopt_long])
AC_COMPILE_IFELSE(
[AC_LANG_PROGRAM([[
#include <stdlib.h>
#include <getopt.h>
struct option long_options[] = {
	{"opt1", required_argument, 0, 'a'},
	{"opt2", optional_argument, 0, 'b'},
	{"opt3",       no_argument, 0, 'c'}
};
void parse_opts(int argc, char **argv) {
	getopt_long(argc, argv, "a:b::c", &long_options[0], &optind);
}
]], [])]
,[AC_MSG_RESULT([yes])]
,[AC_MSG_ERROR([Your compiler does not support \`_GNU_SOURCE\`.])])

dnl Check timeval struct members.
AC_CHECK_MEMBER([struct timeval.tv_sec], [],
    [AC_MSG_ERROR([Invalid \`struct timeval\` structure.])],
    [[#include <sys/time.h>]])
AC_CHECK_MEMBER([struct timeval.tv_usec], [],
    [AC_MSG_ERROR([Invalid \`struct timeval\` structure.])],
	[[#include <sys/time.h>]])

dnl Check size
AC_CHECK_SIZEOF(char)
AC_CHECK_SIZEOF(uint8_t)
AC_CHECK_SIZEOF(uint16_t)
AC_CHECK_SIZEOF(uint32_t)
AC_CHECK_SIZEOF(int)
if test $ac_cv_sizeof_char != "1" -o \
  $ac_cv_sizeof_uint8_t != "1" -o \
  $ac_cv_sizeof_uint16_t != "2" -o \
  $ac_cv_sizeof_uint32_t != "4" -o \
  $ac_cv_sizeof_int != "4"; then
	AC_MSG_ERROR([Invalid type size.])
fi

AC_MSG_CHECKING([for __attribute__ ((packed))])
AC_COMPILE_IFELSE(
[AC_LANG_PROGRAM([[
#include <stdlib.h>
struct foo {
    int num;
    char *str;
    void *ptr;
} __attribute__ ((packed));
]], [])]
,[AC_MSG_RESULT([yes])]
,[AC_MSG_ERROR([Your compiler does not support \`__attribute__ ((packed))\`.])])

dnl Check for std functions.
AC_CHECK_FUNCS([malloc calloc free memcpy memset printf sprintf vsnprintf strerror strlen strncmp strstr strtol strtoul fopen fprintf gettimeofday close fclose exit getopt_long],,
    [AC_MSG_ERROR([Missing essential std functions.])])

dnl `--disable-pcap`: Enabled if found.
AC_ARG_ENABLE([pcap],
    [AS_HELP_STRING([--disable-pcap], [Disable pcap support. (default: enabled if found)])],,[pcap_enabled=yes])
pcap_enabled=$(echo ${pcap_enabled})
case ${pcap_enabled} in
	1|y|yes) pcap_enabled=yes ;;
	''|0|n|no) pcap_enabled= ;;
	*) AC_MSG_ERROR([Unknown option \`${pcap_enabled}\` for --disable-pcap]) ;;
esac

dnl `--disable-selinux`: Enabled if found.
AC_ARG_ENABLE([selinux],
    [AS_HELP_STRING([--disable-selinux], [Disable SELINUX support. (default: enabled if found)])],,[selinux_enabled=yes])
selinux_enabled=$(echo ${selinux_enabled})
case ${selinux_enabled} in
	1|y|yes) selinux_enabled=yes ;;
	''|0|n|no) selinux_enabled= ;;
	*) AC_MSG_ERROR([Unknown option \`${selinux_enabled}\` for --disable-selinux]) ;;
esac

dnl Check libcap headers/functions.
if test x"${pcap_enabled}" != x -a \
        x"${use_msw}" != xyes; then
	AC_CHECK_HEADERS([pcap.h],,
	    [pcap_enabled=])
	AC_SEARCH_LIBS([pcap_lookupnet], [pcap],,
	    [pcap_enabled=],)
	AC_CHECK_FUNCS([pcap_compile pcap_close pcap_setfilter pcap_dispatch],,
	    [pcap_enabled=])
fi

dnl Check for more secure randomization functions
AC_CHECK_HEADERS([bsd/stdlib.h],, [random_enabled=yes])
AC_SEARCH_LIBS([arc4random], [bsd],,,)
AC_CHECK_FUNCS([arc4random], [random_enabled=],)
if test x"${random_enabled}" != x; then
	AC_CHECK_FUNCS([timespec_get srandom random],
	               [random_enabled=yes],
	               [random_enabled=])
else
	arc4random_enabled=yes
fi

dnl Check for SELINUX
if test x"${selinux_enabled}" != x; then
	AC_CHECK_HEADERS([selinux/selinux.h],,
	    [selinux_enabled=])
	AC_SEARCH_LIBS([setcon], [selinux],,[selinux_enabled=],)
fi

dnl Check for ICMP_FILTER
AC_MSG_CHECKING([for working ICMP_FILTER])
AC_COMPILE_IFELSE(
[AC_LANG_PROGRAM([[
#include <netinet/in.h>
#include <sys/socket.h>
#include <linux/icmp.h>
void foo() {
    struct icmp_filter filt;
    int sockfd = socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);
    filt.data  = ~((1<<ICMP_ECHO) | (1<<ICMP_ECHOREPLY));
    setsockopt(sockfd, SOL_RAW, ICMP_FILTER, &filt, sizeof filt);
}
]], [])]
,[AC_MSG_RESULT([yes])
  with_icmp_filter="yes"]
,[AC_MSG_RESULT([no])
  with_icmp_filter="no"])

dnl Check for Android liblog.so
AC_SEARCH_LIBS([__android_log_vprint], [log],,,)

dnl Set automake conf vars
AM_CONDITIONAL([HAVE_PCAP], [test x"${pcap_enabled}" = xyes])
AM_CONDITIONAL([HAVE_SELINUX], [test x"${selinux_enabled}" = xyes])
AM_CONDITIONAL([IS_WINDOWS], [test x"${use_msw}" = xyes])
AM_CONDITIONAL([HAVE_ICMPFILTER], [test x"${with_icmp_filter}" = xyes])
AM_CONDITIONAL([HAVE_RANDOM], [test x"${random_enabled}" = xyes])
AM_CONDITIONAL([HAVE_ARC4RANDOM], [test x"${arc4random_enabled}" = xyes])

dnl output config headers
AC_CONFIG_HEADERS([src/config.h:src/config.h.in])
AC_OUTPUT
