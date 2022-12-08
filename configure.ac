AC_PREREQ([2.65])
m4_define(sr_major_version, 1)
m4_define(sr_minor_version, 0)
m4_define(sr_micro_version, 0)
m4_define(sr_devtag, [dev])
m4_ifnblank(sr_devtag,
	[m4_define(sr_version, sr_major_version.sr_minor_version.sr_micro_version-sr_devtag)],
	[m4_define(sr_version, sr_major_version.sr_minor_version.sr_micro_version)])

# FIXME: can i get a timestamp in here to append to sqsh images?
#        didn't i used to do that somewhere...?

AC_INIT([Source Ruckus Jammy System Builder], [sr_version], [veggiemike@sourceruckus.org], [sourceruckus-jammy])
AC_CONFIG_AUX_DIR([.])
#AC_CONFIG_MACRO_DIR([m4])

AC_SUBST(SR_MAJOR_VERSION, sr_major_version)
AC_SUBST(SR_MINOR_VERSION, sr_minor_version)
AC_SUBST(SR_MICRO_VERSION, sr_micro_version)
AC_SUBST(SR_DEVTAG, sr_devtag)

AM_INIT_AUTOMAKE([dist-xz no-dist-gzip tar-pax foreign subdir-objects color-tests])

AC_PROG_INSTALL
AC_PROG_MKDIR_P
AC_PROG_LN_S

# check for sed /w --in-place
#
# NOTE: Using sed -i in makefiles is frowned upon because it's not
#       portable.  We make 'sed -i' a requirement and check for it here
#       because... I like it and I think pretty much everyone has it.  But
#       I do want to get bonked upside the head if it's going to cause
#       problems.
#
AC_PROG_SED
AS_TMPDIR(sr-)
AC_CACHE_CHECK([whether $ac_cv_path_SED supports -i], [sr_cv_sed_i],
  [AS_IF([touch $tmp/sed && $ac_cv_path_SED -i 's|||' $tmp/sed 2>/dev/null],
      sr_cv_sed_i=yes,
      sr_cv_sed_i=no)])
rm -f $tmp/sed && rmdir $tmp
if test $sr_cv_sed_i != yes; then
    AC_MSG_ERROR([sed -i is required])
fi

# check for needed programs
AC_DEFUN([NEEDED_PROGS],
	 [dnl list them here
	  debootstrap
	  wget
	  bash
	 ])

m4_foreach_w([P], NEEDED_PROGS,
  [AC_PATH_PROG(m4_bpatsubst(m4_toupper(P), [\.], [_]), P)
   AS_IF([test "$m4_bpatsubst(m4_toupper(P), [\.], [_])" = ""],
     AC_MSG_ERROR([Could not find P]))
  ])


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

echo
echo
echo
echo "------------------------------------------------------------------------"
echo "$PACKAGE_STRING"
echo "------------------------------------------------------------------------"
echo
echo "Configuration Options Summary:"
echo
# FIXME: should really generate this list better, potentially with
#       --enable-es-file, --enable-es-web, etc...
#
echo "Enabled Sqsh Images:"
AS_IF([test "x$es_devel" != "xyes"],
[
echo "  es.sqsh es-file.sqsh es-web.sqsh vmm.sqsh workstation.sqsh"
],
[
echo "  es-devel.sqsh"
])
echo
