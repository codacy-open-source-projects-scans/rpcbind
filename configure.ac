AC_INIT(rpcbind, 1.2.7)
 
AM_INIT_AUTOMAKE
AC_CONFIG_SRCDIR([src/rpcbind.c])
AC_PREFIX_DEFAULT(/usr)
AC_PROG_CC

AC_ARG_ENABLE([libwrap],
  AS_HELP_STRING([--enable-libwrap], [Enables host name checking through tcpd @<:@default=no@:>@]))
AM_CONDITIONAL(LIBWRAP, test x$enable_libwrap = xyes)

AC_ARG_ENABLE([debug],
  AS_HELP_STRING([--enable-debug], [Turns on rpcbind debugging @<:@default=no@:>@]))
AM_CONDITIONAL(DEBUG, test x$enable_debug = xyes)
if test x"$enable_debug" = xyes; then
	AC_CHECK_LIB([tirpc], [libtirpc_set_debug], [lib_setdebug=yes])
fi
AM_CONDITIONAL(LIBSETDEBUG, test x$lib_setdebug = xyes)

AC_ARG_ENABLE([warmstarts],
  AS_HELP_STRING([--enable-warmstarts], [Enables Warm Starts @<:@default=no@:>@]))
AM_CONDITIONAL(WARMSTART, test x$enable_warmstarts = xyes)
AS_IF([test x$enable_warmstarts = xyes], [
	warmstarts_opt=-w
], [
	warmstarts_opt=
])
AC_SUBST([warmstarts_opt], [$warmstarts_opt])

AC_ARG_ENABLE([rmtcalls],
  AS_HELP_STRING([--enable-rmtcalls], [Enables Remote Calls @<:@default=no@:>@]))
AM_CONDITIONAL(RMTCALLS, test x$enable_rmtcalls = xyes)

AC_ARG_WITH([statedir],
  AS_HELP_STRING([--with-statedir=ARG], [use ARG as state dir @<:@default=/run/rpcbind@:>@])
  ,, [with_statedir=/run/rpcbind])
AC_SUBST([statedir], [$with_statedir])

AC_ARG_WITH([rpcuser],
  AS_HELP_STRING([--with-rpcuser=ARG], [use ARG for RPC @<:@default=root@:>@])
  ,, [with_rpcuser=root])
AC_SUBST([rpcuser], [$with_rpcuser])
 
AC_ARG_WITH([nss_modules],
  AS_HELP_STRING([--with-nss-modules=NSS_MODULES]
  , [Sets the nss module search list to the given space-delimited string.
     For example --with-nss-modules="files altfiles" @<:@default=files@:>@])
  ,, [with_nss_modules=files])
AC_SUBST([nss_modules], [$with_nss_modules])

PKG_CHECK_MODULES([TIRPC], [libtirpc])

CPPFLAGS=$TIRPC_CFLAGS
AC_MSG_CHECKING([for abstract socket support in libtirpc])
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([
#include <rpc/rpc.h>
],[
char *path = _PATH_RPCBINDSOCK_ABSTRACT;
])], [have_abstract=yes], [have_abstract=no])
CPPFLAGS=
AC_MSG_RESULT([$have_abstract])
AM_CONDITIONAL(ABSTRACT, [ test "x$have_abstract" = "xyes" ])

PKG_PROG_PKG_CONFIG
AC_ARG_WITH([systemdsystemunitdir],
  AS_HELP_STRING([--with-systemdsystemunitdir=DIR], [Directory for systemd service files]),
  [], [with_systemdsystemunitdir=$($PKG_CONFIG --variable=systemdsystemunitdir systemd)])
  if test "x$with_systemdsystemunitdir" != xno; then
    AC_SUBST([systemdsystemunitdir], [$with_systemdsystemunitdir])
     PKG_CHECK_MODULES([SYSTEMD], [libsystemd], [],
	   [PKG_CHECK_MODULES([SYSTEMD], [libsystemd-daemon], [],
	   AC_MSG_ERROR([libsystemd support requested but found]))])
  fi
AM_CONDITIONAL(SYSTEMD, [test -n "$with_systemdsystemunitdir" -a "x$with_systemdsystemunitdir" != xno ])

AS_IF([test x$enable_libwrap = xyes], [
	AC_CHECK_LIB([wrap], [hosts_access], ,
		AC_MSG_ERROR([libwrap support requested but unable to find libwrap]))
])

AC_SEARCH_LIBS([pthread_create], [pthread])

AC_CHECK_HEADERS([nss.h])

# make sbindir available for substitution in config file
# 2 "evals" needed to expand variable names
AC_SUBST([_sbindir])
AC_CONFIG_COMMANDS_PRE([eval eval _sbindir=$sbindir])

AC_OUTPUT([Makefile systemd/rpcbind.service systemd/rpcbind.socket])
