# Process this file with autoconf to produce a configure script.

AC_INIT([irrd], m4_esyscmd_s(echo $(git describe --abbrev=4 --dirty --always --tags | sed 's/^v//')))
AM_INIT_AUTOMAKE([foreign])

AC_PREREQ([2.59])
AC_CONFIG_SRCDIR([lib/mrt/mrt.c])
AC_CONFIG_HEADERS(include/config.h)
AC_CONFIG_MACRO_DIR([m4])
AC_INCLUDES_DEFAULT

# Checks for programs.
AC_PROG_CC

# what OS are we?
#AC_CANONICAL_HOST

# initializing defaults
# I've seen a report that pthread support is shaky in FreeBSD 2.x releases
case "${host_os}" in
    freebsd2* )
	disable_thread=yes
    ;;
    * ) 
	disable_thread=no
    ;;
esac

AC_ARG_ENABLE(thread,
[  --disable-thread        disable thread always],
[disable_thread=yes],)

AC_ARG_ENABLE(thread,
[  --enable-thread         enable thread (on non-solaris platform)],
[disable_thread=no],)

AC_ARG_ENABLE(wall,
[  --disable-wall          disable -Wall option always],
[disable_wall=yes],)

pwd=`pwd`
pwd=`basename $pwd`
if test "$pwd" = "src"; then
    os=${host_os}
else
    os=`expr $pwd : 'src\.\(.*\)'`
fi

HOST_OS="$os"
dnl AC_SUBST(exec_prefix)
AC_SUBST(HOST_OS)

dnl set default of no support in case no support
dnl DESTDIR=/usr/local/sbin
dnl MANDIR=/usr/local/man
MODULES=""

EXTRA_INCLUDE_DIRS=""
EXTRA_LIBS=""
CPPFLAGS="-D_FILE_OFFSET_BITS=64 -DFUNCPROTO -I../../include"
dnl defining FUNCPROTO ensures that struct library uses portotype
dnl we don't expect non-ANSI (K&R) C compilers to be used
dnl Some compiles may define __STDC__ by itself which makes it enable, 
dnl but struct library also requires its values is non-zero

AC_PROG_CC
AC_PROG_CPP
dnl AC_PROC_CC may set an intial CFLAGS depending on whether it accepts "-g".
dnl if test "$GCC" = "yes"; then
dnl    CFLAGS="-g -O"
dnl fi

AC_MSG_CHECKING(for -Wall option)
if test "$disable_wall" = yes; then
   if test "$GCC" != "yes"; then
dnl shut up SUN WorkShop Compilers
       CFLAGS="$CFLAGS -w"
   fi
   AC_MSG_RESULT(disabled)
elif test "$GCC" = "yes"; then
dnl "-pedantic" was too strict to run on netbsd and linux
dnl   CFLAGS="$CFLAGS -pedantic -Wall"
   CFLAGS="$CFLAGS -Wall"
   AC_MSG_RESULT(yes)
else
   AC_MSG_RESULT(no)
fi
QUANTIFY="quantify"
PURIFY="purify"

AC_DEFUN([AC_FILE_EXIST], [
if test -f "$1" >/dev/null 2>&1; then
  ifelse([$2], , :, [$2])
  ifelse([$3], , , [else $3])
fi])

case "${host_os}" in
    *solaris* )
dnl	XPG4 support shouldn't be necessary -ljb
dnl	AC_MSG_CHECKING(for Solaris XPG4 support)
dnl	if test -f /usr/lib/libxnet.so; then
dnl	    CPPFLAGS="$CPPFLAGS -D_POSIX_C_SOURCE=199506 -D_XOPEN_SOURCE=500"
dnl	    CPPFLAGS="$CPPFLAGS -D_XOPEN_SOURCE_EXTENDED=1 -D_XOPEN_VERSION=4"
dnl	    CPPFLAGS="$CPPFLAGS -D__EXTENSIONS__ -D_VA_LIST"
dnl	    LIBS="$LIBS -lxnet /usr/ccs/lib/values-xpg4.o"
dnl  	    AC_MSG_RESULT(yes)
dnl	else
dnl  	    AC_MSG_RESULT(no)
dnl	fi
	if test "$GCC" = "yes"; then
	    CFLAGS="$CFLAGS -Wno-char-subscripts"
	fi
    ;;
esac

case "${host_os}" in
    solaris* )
	CRYPT_LIB="-lcrypt"
    ;;
    bsdi* )
        LIBS="$LIBS -lipc"
    ;;
    freebsd*|netbsd*|openbsd* )
        if test "$disable_thread" != yes; then
		LDFLAGS="$LDFLAGS -pthread"
                THREAD_CFLAGS="-D_THREAD_SAFE -pthread"
        fi
	CRYPT_LIB="-lcrypt"
        ac_pthrlib="c_r"
    ;;
    linux* )
	CRYPT_LIB="-lcrypt"
    ;;
    *86*-darwin*)
    ;;
    osf* )
	if test $CC = "cc" ; then
           CPPFLAGS="$CPPFLAGS -taso -pthread"
        fi
        if test "$disable_thread" != yes; then
		LDFLAGS="$LDFLAGS -lc_r"
                THREAD_CFLAGS="-D_THREAD_SAFE"
	fi
        CPPFLAGS="$CPPFLAGS -L/usr/shlib"
        ac_pthrlib="pthreads"
    ;;

esac

dnl Checks for programs.
AC_PROG_MAKE_SET
dnl because it will be executed in program/*
ac_install_sh=../../$ac_install_sh
AC_PROG_INSTALL
AC_PROG_RANLIB
AC_CHECK_PROGS(MAKEDEP, makedepend mkdep)
if test "$MAKEDEP" = "makedepend"; then
    MAKEDEPCLEAN="rm -f .depend"
fi
AC_SUBST(MAKEDEPCLEAN)

dnl Checks for pthread libraries. (linux has pthread.h only, so ...)
if test "$disable_thread" = yes; then
  AC_MSG_CHECKING(for thread support)
  AC_MSG_RESULT(disabled)
else
  if test ! -n "$ac_pthrlib"; then 
    ac_pthrlib="pthread"
  fi;
  AC_CHECK_LIB($ac_pthrlib, pthread_create, [
    AC_CHECK_HEADERS(pthread.h, [pthread="yes"])])
fi

AC_MSG_CHECKING(for thread support)
if test "$pthread" = yes; then
  AC_MSG_RESULT(yes)
  AC_DEFINE(HAVE_LIBPTHREAD)
  CFLAGS="$CFLAGS $THREAD_CFLAGS"
  CPPFLAGS="-D_REENTRANT -D_POSIX_PTHREAD_SEMANTICS $CPPFLAGS"
  QUANTIFY="$QUANTIFY -threads=yes -max-threads=80"
  PURIFY="$PURIFY -threads=yes -max-threads=80"
  dnl Check for gthreads
  PKG_CHECK_MODULES(GTHREADS, gthread-2.0 >= 2.6,  AC_MSG_RESULT([found], AC_MSG_ERROR([gthread-2.0 >= 2.6 not found])))
  AC_SUBST(GTHREADS_CFLAGS)
  AC_SUBST(GTHREADS_LIBS)
  THREADLIBS="-l$ac_pthrlib"
  LIBS="$LIBS $THREADLIBS"
dnl at least FreeBSD 3.1 doesn't have this
  AC_CHECK_LIB($ac_pthrlib, pthread_attr_setscope, 
	       AC_DEFINE(HAVE_PTHREAD_ATTR_SETSCOPE))
  AC_CHECK_FUNCS(thr_setconcurrency)
else
  AC_MSG_RESULT(no)
fi

dnl Checks for various functions

AC_CHECK_FUNCS(gethostbyname)
if test x"$ac_cv_func_gethostbyname" = x"no"; then
AC_CHECK_LIB(nsl, gethostbyname)
fi

AC_CHECK_FUNCS(socket)
if test x"$ac_cv_func_socket" = x"no"; then
AC_CHECK_LIB(socket, socket)
fi

AC_CHECK_FUNCS(inet_ntop)
if test x"$ac_cv_func_inet_ntop" = x"no"; then
AC_CHECK_LIB(resolv, inet_ntop, [
  LIBS="$LIBS -lresolv"
  AC_DEFINE(HAVE_INET_NTOP)
  AC_CHECK_HEADERS(resolv.h)]
)
fi

dnl if test -d /usr/local/include; then
dnl     CPPFLAGS="$CPPFLAGS -I/usr/local/include"
dnl fi
dnl if test -d /usr/local/lib; then
dnl    LDFLAGS="$LDFLAGS -L/usr/local/lib"
dnl fi

dnl find flex
AC_PATH_PROG(FLEX_PATH,flex,no)
if test $FLEX_PATH = no; then
 AC_MSG_ERROR([flex is required -- see www.gnu.org/software/flex/])
fi

dnl find bison/yacc
AC_PROG_YACC
dnl AC_PATH_PROG(BISON_PATH,bison,no)
dnl if test $BISON_PATH = no; then
dnl    BISON_PATH=""
dnl    echo "** No BISON -- only IRRD will be built! **"
dnl fi

AC_SUBST(FLEX_PATH)

dnl sendmail mail and pgp
AC_PATH_PROG(SENDMAIL_PATH,sendmail,no,$PATH:/usr/lib)
if test $SENDMAIL_PATH = no; then
    SENDMAIL_PATH=""
else
    AC_DEFINE(HAVE_SENDMAIL)
fi
AC_PATH_PROG(MAIL_PATH,mail,no)
if test $MAIL_PATH = no; then
    MAIL_PATH=""
else
    AC_DEFINE(HAVE_MAIL)
fi

dnl Prefer GPG, second choice PGP version 5

AC_ARG_WITH(pgpv5,
[  --with-pgpv5            Prefer PGP version 5 over the default Gnu PGP (GPG)],
[ case "$withval" in
  no)
    ac_pgpv5=no
  ;;
  *)
    ac_pgpv5=yes
  ;;
  esac ],
[ac_pgpv5=undef]
)

if test $ac_pgpv5 != yes; then
	AC_PATH_PROG(GPG_PATH,gpg,no)
else
	GPG_PATH=no
fi

if test $GPG_PATH = no -o $ac_pgpv5 = yes; then
	if test $ac_pgpv5 = no; then
		PGPK_PATH=no
		PGPS_PATH=no
		PGPV_PATH=no
	else
		AC_PATH_PROG(PGPK_PATH,pgpk,no)
		AC_PATH_PROG(PGPS_PATH,pgps,no)
		AC_PATH_PROG(PGPV_PATH,pgpv,no)
	fi
	if test $PGPK_PATH = no -o $PGPS_PATH = no -o $PGPV_PATH = no; then
		PGPK_PATH=""
		PGPS_PATH=""
		PGPV_PATH=""
		GPG_PATH=""
		PGPDIR=gpg
	else	
		GPG_PATH=""
		PGPDIR=pgpv5
		AC_DEFINE(PGP)
	fi
else
	PGPK_PATH=""
	PGPS_PATH=""
	PGPV_PATH=""
	PGPDIR=gpg
	AC_DEFINE(PGP)
fi
AC_SUBST(PGP)
AC_SUBST(PGPDIR)
AC_SUBST(GPG_PATH)
AC_SUBST(PGPV_PATH)
AC_SUBST(PGPK_PATH)
AC_SUBST(PGPS_PATH)

AC_PATH_PROG(GZIP_PATH,gzip,no)
if test $GZIP_PATH = no; then
    GZIP_PATH=""
else
    AC_DEFINE(HAVE_GZIP)
fi
AC_PATH_PROG(IRRDCACHER_PATH,irrdcacher,no)
if test $IRRDCACHER_PATH = no; then
    IRRDCACHER_PATH=""
else
    AC_DEFINE(HAVE_IRRDCACHER)
fi
AC_PATH_PROG(WGET_PATH,wget,no)
if test $WGET_PATH = no; then
    WGET_PATH=""
else
    AC_DEFINE(HAVE_WGET)
fi
AC_SUBST(SENDMAIL_PATH)
AC_SUBST(MAIL_PATH)
AC_SUBST(GZIP_PATH)
AC_SUBST(IRRDCACHER_PATH)
AC_SUBST(WGET_PATH)

dnl Checks for header files.
AC_HEADER_STDC

dnl Checks for functions.
AC_FUNC_SETPGRP	

dnl Checks for alloca()
AC_FUNC_ALLOCA

AC_CHECK_HEADERS(sys/stropts.h sys/select.h sys/time.h)
AC_CHECK_HEADERS(libgen.h)

dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_HEADER_TIME
AC_STRUCT_TM

dnl u_char, u_int, u_short, and u_long
AC_MSG_CHECKING([u_char, u_int, u_short, and u_long])
  AC_COMPILE_IFELSE([AC_LANG_PROGRAM([[#include <sys/types.h>]], [[
  u_char c; u_int i; u_short s; u_long l;]])],[AC_DEFINE(HAVE_U_TYPES) AC_MSG_RESULT(yes)],[AC_MSG_RESULT(no)]);

dnl Check for in6_addr structure definition
AC_EGREP_HEADER(struct in6_addr, netinet/in.h,
                  AC_DEFINE(HAVE_STRUCT_IN6_ADDR))

dnl Checks for library functions.
AC_CHECK_FUNCS(localtime_r strtok_r gethostbyname_r gethostbyaddr_r)
AC_CHECK_FUNCS(sigaction sigset)
AC_CHECK_FUNCS(sigprocmask)
AC_CHECK_FUNCS(memmove)
AC_CHECK_FUNCS(strerror)
AC_CHECK_FUNCS(setsid)
AC_CHECK_FUNCS(mkstemp)
AC_CHECK_FUNCS(mkdtemp)
AC_CHECK_FUNCS(snprintf)
AC_CHECK_FUNCS(dirname basename)

dnl Check for glib
PKG_CHECK_MODULES(GLIB, glib-2.0 >= 2.6, AC_MSG_RESULT([found], AC_MSG_ERROR([glib-2.0 >= 2.6 not found])))
AC_SUBST(GLIB_CFLAGS)
AC_SUBST(GLIB_LIBS)

dnl Check for postgres

dnl ---------------------------------------------------------------------------
dnl Macro: AC_CHECK_PGSQL
dnl First check for custom PostgreSQL paths in --with-pgsql-* options.
dnl If some paths are missing,  check if pg_config exists. 
dnl ---------------------------------------------------------------------------

AC_DEFUN([AC_CHECK_PGSQL],[

# Check for custom includes path
if test [ -z "$ac_cv_pgsql_includes" ]
then 
	AC_ARG_WITH([pgsql-includes],
		AC_HELP_STRING([--with-pgsql-includes], [path to PostgreSQL header files]),
		[ac_cv_pgsql_includes=$withval])
fi
if test [ -n "$ac_cv_pgsql_includes" ]
then
	AC_CACHE_CHECK([PostgreSQL includes], [ac_cv_pgsql_includes], [ac_cv_pgsql_includes=""])
	PGSQL_CFLAGS="-I$ac_cv_pgsql_includes"
fi

# Check for custom library path
if test [ -z "$ac_cv_pgsql_libs" ]
then
	AC_ARG_WITH([pgsql-libs],
		AC_HELP_STRING([--with-pgsql-libs], [path to PostgreSQL libraries]),
		[ac_cv_pgsql_libs=$withval])
fi
if test [ -n "$ac_cv_pgsql_libs" ]
then
	AC_CACHE_CHECK([PostgreSQL libraries], [ac_cv_pgsql_libs], [ac_cv_pgsql_libs=""])
	PGSQL_LIBS="-L$ac_cv_pgsql_libs -lpq"
fi

# If some path is missing, try to autodetermine with pgsql_config
if test [ -z "$ac_cv_pgsql_includes" -o -z "$ac_cv_pgsql_libs" ]
then
    if test [ -z "$pgconfig" ]
    then 
        AC_PATH_PROG(pgconfig,pg_config)
    fi
    if test [ -z "$pgconfig" ]
    then
        AC_MSG_ERROR([pg_config executable not found
********************************************************************************
ERROR: cannot find PostgreSQL libraries. If you want to compile with PosgregSQL support,
       you must either specify file locations explicitly using 
       --with-pgsql-includes and --with-pgsql-libs options, or make sure path to 
       pg_config is listed in your PATH environment variable. If you want to 
       disable PostgreSQL support, use --without-pgsql option.
********************************************************************************
])
    else
        if test [ -z "$ac_cv_pgsql_includes" ]
        then
            AC_MSG_CHECKING(PostgreSQL C flags)
            PGSQL_CFLAGS="-I`${pgconfig} --includedir`"
            AC_MSG_RESULT($PGSQL_CFLAGS)
        fi
        if test [ -z "$ac_cv_pgsql_libs" ]
        then
            AC_MSG_CHECKING(PostgreSQL linker flags)
            PGSQL_LIBS="-L`${pgconfig} --libdir` -lpq"
            AC_MSG_RESULT($PGSQL_LIBS)
        fi
    fi
fi
])


ac_postgres=false
AC_MSG_CHECKING(for postgres)
AC_ARG_WITH(postgres,
	[  --with-postgres ],
	[AC_MSG_RESULT("$with_postgres")
		case $with_postgres in
		yes)
			AC_CHECK_PGSQL()
			CFLAGS="$PGSQL_CFLAGS $CFLAGS"
			LDFLAGS="$PGSQL_LIBS $LDFLAGS"
			CFLAGS="-DOPT_POSTGRES $CFLAGS"
			;;
		*)
			;;
		esac
	],
	[AC_MSG_RESULT([--with-postgres not specified])]
	)

AC_CHECK_DECLS(utimes,,,[#include <sys/time.h>])
AC_CHECK_DECLS(inet_ntop,,,[#include <netinet/in.h>
#include <sys/types.h>
#include <arpa/inet.h>
#include <arpa/nameser.h>
#include <resolv.h>])
AC_CHECK_DECLS(mkstemp)
AC_CHECK_DECLS(mkdtemp)
AC_CHECK_DECLS(getdtablesize)

AC_SUBST(DB_LIB)
AC_SUBST(DESTDIR)
AC_SUBST(MANDIR)
AC_SUBST(MODULES)
AC_SUBST(IRRD_OPS_LIBS)

AC_SUBST(CFLAGS)
AC_SUBST(CPPFLAGS)
AC_SUBST(DEFINES)
AC_SUBST(EXTRA_INCLUDE_DIRS)
AC_SUBST(EXTRA_LIBS)
AC_SUBST(QUANTIFY)
AC_SUBST(PURIFY)

AC_SUBST(CRYPT_LIB)
AC_SUBST(THREADLIBS)
AC_CONFIG_FILES([Makefile Make.include:../src/Make.include.in include/pipeline_defs.h include/irr_defs.h])
AC_OUTPUT
