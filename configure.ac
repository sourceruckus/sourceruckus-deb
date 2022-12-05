AC_PREREQ([2.65])
AC_INIT([sourceruckus-deb], [0.0.0-dev], [veggiemike@sourceruckus.org])

AM_INIT_AUTOMAKE([dist-xz no-dist-gzip foreign])

AC_PROG_INSTALL
AC_PROG_MKDIR_P
AC_PROG_LN_S

# FIXME: check for debootstrap
# FIXME: check sudo skills or that we're running as root

# build a devel-enabled base image
AC_ARG_ENABLE(devel,
  AS_HELP_STRING([--enable-devel], [Enable building es-devel.sqsh [default=no]]),
  es_devel=$enableval,
  es_devel=no)
AC_SUBST(ES_DEVEL, $es_devel)
AM_CONDITIONAL([ES_DEVEL], [test x$ES_DEVEL = xyes])

AC_CONFIG_FILES([Makefile
		 ])
AC_OUTPUT
