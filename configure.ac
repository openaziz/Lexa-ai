dnl configure.ac -- autoconf template for nmh
dnl

AC_PREREQ([2.68])

AC_INIT([nmh], m4_normalize(m4_include([VERSION])), [nmh-workers@nongnu.org])
AC_CONFIG_SRCDIR([h/nmh.h])
AC_CONFIG_HEADERS([config.h])
AM_INIT_AUTOMAKE([-Wall foreign serial-tests subdir-objects 1.12])

AC_CANONICAL_HOST

AC_MSG_NOTICE([configuring for AC_PACKAGE_NAME-AC_PACKAGE_VERSION])
AC_SUBST([VERSION], [AC_PACKAGE_VERSION])dnl

dnl What date of nmh are we building?
DATE=`cat ${srcdir}/DATE`
AC_MSG_NOTICE([configuring for nmh dated $DATE])
AC_SUBST([DATE])dnl

dnl --------------------------
dnl CHECK COMMAND LINE OPTIONS
dnl --------------------------

dnl Do you want client-side support for using SASL for authentication?
dnl Note that this code will be enabled for both POP and SMTP
AC_ARG_WITH([cyrus-sasl], AS_HELP_STRING([--with-cyrus-sasl],
  [Enable SASL support via the Cyrus SASL library]))
AS_IF([test x"$with_cyrus_sasl" != x -a x"$with_cyrus_sasl" != xyes -a \
            x"$with_cyrus_sasl" != xno],[
  AC_MSG_WARN([Arguments to --with-cyrus-sasl now ignored])
  AC_MSG_WARN([Please pass the appropriate arguments to CPPFLAGS/LDFLAGS])])

dnl Do you want client-side support for encryption with TLS?
AC_ARG_WITH([tls], AS_HELP_STRING([--with-tls], [Enable TLS support]))

dnl Do you want client-side support for using OAuth2 for SMTP & POP auth?
AC_ARG_WITH([oauth], AS_HELP_STRING([--with-oauth],
  [Enable OAuth2 support in SMTP and POP auth]))

dnl Set the backup prefix
AC_ARG_WITH([hash-backup],
  AS_HELP_STRING([--with-hash-backup],[use # as the backup prefix (default: ,)]))
AS_IF([test x"$with_hash_backup" != x -a x"$with_hash_backup" != x"no"],
      [backup_prefix="#"], [backup_prefix=","])
AC_DEFINE_UNQUOTED([BACKUP_PREFIX], "$backup_prefix",
    [The prefix that is prepended to the name of message files when they are "removed" by rmm. This should typically be `,' or `#'.])dnl

dnl What method of posting should post use?
AC_ARG_WITH([mts],
  AS_HELP_STRING([--with-mts=@<:@smtp|sendmail/smtp|sendmail/pipe@:>@],
  [specify the default mail transport agent/service]))

AS_IF([test x"$with_mts" = x"smtp"], [MTS="smtp"],
      [test x"$with_mts" = x"sendmail"], [MTS="sendmail/smtp"],
      [test x"$with_mts" = x"sendmail/smtp"], [MTS="sendmail/smtp"],
      [test x"$with_mts" = x"sendmail/pipe"], [MTS="sendmail/pipe"],
      [MTS="smtp"])
AC_SUBST([MTS])dnl

dnl What should be the default mail server?
AC_ARG_WITH([smtpserver],
  [AS_HELP_STRING([--with-smtpserver='SMTPSERVER'],
  [specify the default SMTP server @<:@localhost@:>@])])
AS_IF([test -n "$with_smtpserver"], [smtpserver="$with_smtpserver"],
      [smtpserver="localhost"])
AC_SUBST([smtpserver])dnl

dnl -------------------------------------------------------------------
dnl Default location is /usr/local/nmh/{bin,etc,libexec,man}, unless we
dnl find an existing installation, in which case we use its location.
dnl -------------------------------------------------------------------
AC_PREFIX_DEFAULT([/usr/local/nmh])
AC_PREFIX_PROGRAM([mhparam])

dnl ------------------
dnl CHECK THE COMPILER
dnl ------------------

AC_PROG_CC([cc gcc])
AM_PROG_CC_C_O

dnl ------------------
dnl CHECK HEADER FILES
dnl ------------------

AC_HEADER_TIOCGWINSZ
AC_CHECK_HEADERS([fcntl.h stdbool.h wchar.h wctype.h \
                  sys/param.h sys/time.h sys/stream.h])

AC_CHECK_HEADER([sys/ptem.h], AC_DEFINE(WINSIZE_IN_PTEM,1,
  [Define to 1 if `struct winsize' requires <sys/ptem.h>.]),,
[[#if HAVE_SYS_STREAM_H
#  include <sys/stream.h>
#endif
]])

dnl ---------------
dnl CHECK FUNCTIONS
dnl ---------------
AC_CHECK_FUNCS([wcwidth mbtowc getutxent arc4random mkstemps])

dnl Use custom getline for platforms that don't have it.
AC_CONFIG_LIBOBJ_DIR([sbr])
AC_REPLACE_FUNCS([getline])

dnl -----------------------
dnl CHECK MULTIBYTE SUPPORT
dnl -----------------------
AS_IF([test "x$ac_cv_header_wchar_h" = "xyes" -a \
	    "x$ac_cv_header_wctype_h" = "xyes" -a \
	    "x$ac_cv_func_wcwidth" = "xyes" -a \
	    "x$ac_cv_func_mbtowc" = "xyes"],
      [AC_DEFINE([MULTIBYTE_SUPPORT], [1],
		 [Define to enable support for multibyte character sets.])
       MULTIBYTE_ENABLED=1],
      [MULTIBYTE_ENABLED=0])
AC_SUBST([MULTIBYTE_ENABLED])

dnl ----------------
dnl CUSTOMIZE CFLAGS
dnl ----------------
dnl The user didn't specify CFLAGS, so customize them.

if test "$GCC" != yes; then
  dnl Use -v and some other options with SunStudio cc.  lex produces
  dnl code that causes unreachable-statement warnings.  It might be
  dnl better to use an autoconf test, except -v will probably succeed
  dnl with many other compilers but have different meaning.
  AS_CASE([`${CC} -V 2>&1`],
          [cc:*\ Sun\ C*], [CFLAGS=\
"${CFLAGS:+$CFLAGS }-v -errtags=yes -erroff=E_STATEMENT_NOT_REACHED"])
fi

AC_CACHE_CHECK([whether compiler supports -Wall], [nmh_cv_has_wall],
               [nmh_saved_cflags="$CFLAGS"
               CFLAGS="$AM_CFLAGS $CFLAGS -Wall -Werror"
               AC_COMPILE_IFELSE([AC_LANG_PROGRAM([],[])],nmh_cv_has_wall=yes,nmh_cv_has_wall=no)
               CFLAGS="$nmh_saved_cflags"])
test "$nmh_cv_has_wall" = "yes"  &&  CFLAGS="${CFLAGS:+$CFLAGS }-Wall"

AC_CACHE_CHECK([whether compiler supports -Wextra], [nmh_cv_has_wextra],
               [nmh_saved_cflags="$CFLAGS"
               CFLAGS="$AM_CFLAGS $CFLAGS -Wextra -Werror"
               AC_COMPILE_IFELSE([AC_LANG_PROGRAM([],[])],nmh_cv_has_wextra=yes,nmh_cv_has_wextra=no)
               CFLAGS="$nmh_saved_cflags"])
test "$nmh_cv_has_wextra" = "yes"  &&  CFLAGS="${CFLAGS:+$CFLAGS }-Wextra"

AC_SUBST([AM_CFLAGS])

dnl Check for any platform-specific additional AM_CPPFLAGS.  This depends on
dnl MULTIBYTE_ENABLED having already been set.
NMH_ADDL_CPPFLAGS

dnl FreeBSD needs a bit of magic to bring getline() into scope.
dnl We do this here rather than in (say) h/mh.h because this macro must
dnl be defined before <stdio.h> is pulled in.
dnl
dnl And while we're here, add the packages tree to the cpp and ld search
dnl paths.  Note that FreeBSD's pkg(8) seems to be hardwired to use /usr/local.
dnl If /usr/ports is installed, we use its idea of where things are installed,
dnl otherwise we assume /usr/local.

AS_IF([test `uname` = FreeBSD],
	[CPPFLAGS="${CPPFLAGS:+$CPPFLAGS }-D_WITH_GETLINE"
	 nmh_cv_freebsd_localbase=`echo '.include <bsd.port.mk>' \
		| make -k -f /dev/stdin -V LOCALBASE 2>/dev/null | sed 1q`
	 test -z "$nmh_cv_freebsd_localbase" && nmh_cv_freebsd_localbase=/usr/local
	 CPPFLAGS="${CPPFLAGS:+$CPPFLAGS }-I$nmh_cv_freebsd_localbase/include"
	 LDFLAGS="${LDFLAGS:+$LDFLAGS }-L$nmh_cv_freebsd_localbase/lib"
])

dnl --------------
dnl CUSTOMIZE LINK
dnl --------------
AS_IF([test `uname` = OpenBSD],
      dnl Munge "often/almost always misused" warnings from OpenBSD linker
      dnl so that they don't color the waterfall.
      [POSTLINK="2>&1 | sed -e \
's/: w\(arning: s.*() is .* misused, please use\)/: W\1/'"])
AC_SUBST([POSTLINK])

dnl -----------------
dnl CUSTOMIZE LDFLAGS
dnl -----------------
dnl Disable clang complaint about unused -ansi when linking.
AC_CACHE_CHECK([whether linker supports -Qunused-arguments],
               [nmh_cv_has_q_unused_arguments],
               [AS_IF([test "$GCC" = yes  &&  `${CC} --version 2>&1 | \
                       grep clang >/dev/null`],
                      [nmh_saved_ldflags="$LDFLAGS"
                       LDFLAGS="$AM_LDFLAGS $LDFLAGS -Qunused-arguments"
                       AC_LINK_IFELSE([AC_LANG_PROGRAM([],[])],nmh_cv_has_q_unused_arguments=yes,
                                      nmh_cv_has_q_unused_arguments=no)
                       LDFLAGS="$nmh_saved_ldflags"],
                       [nmh_cv_has_q_unused_arguments=no])])
test "$nmh_cv_has_q_unused_arguments" = "yes"  &&  \
  AM_LDFLAGS="${AM_LDFLAGS:+$AM_LDFLAGS }-Qunused-arguments"
AC_SUBST([AM_LDFLAGS])

AC_HEADER_ASSERT

dnl ------------------
dnl CHECK FOR PROGRAMS
dnl ------------------
AC_PROG_MAKE_SET        dnl Does make define $MAKE
AC_PROG_INSTALL         dnl Check for BSD compatible `install'
AC_PROG_RANLIB          dnl Check for `ranlib'
AC_PROG_AWK             dnl Check for mawk,gawk,nawk, then awk
AC_PROG_SED             dnl Check for Posix-compliant sed
AC_PROG_YACC            dnl Check for yacc/bison
dnl AM_PROG_LEX doesn't yet have the [no]yywrap argument
AC_PROG_LEX(noyywrap)   dnl Check for lex/flex

AM_PROG_AR

dnl Look for `cut'
pathtmp=/usr/xpg4/bin:/usr/bin:/bin:/usr/local/bin:/usr/ucb
AC_PATH_PROG([cutpath], [cut], [no], [$pathtmp])

dnl
dnl Check for MD5 program and formatting command
dnl
AC_CHECK_PROGS([MD5SUM], [md5sum md5], [missing])
AS_CASE(["${MD5SUM}"],
	[md5sum], [MD5FMT="cat"],
	[md5], [[MD5FMT="${SED} -e 's/MD5 *(.*) *= \([0-9a-f]*\)/\1/'"]],
	[MD5FMT="missing"])
AC_SUBST([MD5FMT])

dnl Look for `ls'
pathtmp=/usr/xpg4/bin:/usr/bin:/bin:/usr/local/bin:/usr/ucb
AC_PATH_PROG([lspath], [ls], [no], [$pathtmp])

dnl See how we get ls to display the owner and the group
AS_IF([test "$lspath" != "no"],
  [AC_CACHE_CHECK([how to get ls to show us the group ownership of a file],
		 [nmh_cv_ls_grpopt],
  [AS_IF([test x"`$lspath -dl / | $AWK '{print $9}'`" = x"/"],[
    dnl There were 9 parameters, so unless this is a really bizarre, nonstandard
    dnl ls, it would seem -l gave us both the user and group.  On this type of
    dnl ls, -g makes _only_ the group be displayed (and not the user).
    nmh_cv_ls_grpopt="-l"],[
    dnl Looks like -l only gave us the user, so we need -g to get the group too.
    nmh_cv_ls_grpopt="-lg"])])])

dnl Look for `sendmail'
pathtmp=/usr/sbin:/usr/lib:/usr/etc:/usr/ucblib:/usr/bin:/bin
AC_PATH_PROG([sendmailpath], [sendmail], [/usr/sbin/sendmail], [$pathtmp])

dnl Cygwin FAT filesystems do not support hard links.  So default to
dnl cp instead, even if running on an NTFS or other filesystem.  (And
dnl therefore, this cannot be made into a dynamic test, in order to
dnl support the least common Cygwin denominator.
AS_CASE(["$host_os"],
  [cygwin*], [LN=cp],
  [LN=ln])
AC_SUBST([LN])

dnl ----------------------------------------------------------
dnl FIND MAIL SPOOL AND SEE IF WE NEED TO MAKE inc SETGID MAIL
dnl ----------------------------------------------------------
AC_CACHE_CHECK(where mail spool is located, nmh_cv_mailspool,
[for mailspool in /var/mail        dnl
                  /var/spool/mail  dnl
                  /usr/spool/mail  dnl
                  /dev/null;       dnl Just in case we fall through
do
  test -d $mailspool && break
done
nmh_cv_mailspool=$mailspool
])
mailspool=$nmh_cv_mailspool
AC_SUBST([mailspool])dnl

dnl See whether the mail spool directory is world-writable.
if test "$lspath" != "no" -a "$cutpath" != "no"; then
  AC_CACHE_CHECK(whether the mail spool is world-writable,
		 nmh_cv_mailspool_world_writable,
  [if test "`$lspath -dlL $mailspool | $cutpath -c9`" = "-"; then
    nmh_cv_mailspool_world_writable=no
  else
    nmh_cv_mailspool_world_writable=yes
  fi])
fi

dnl Also, check for liblockfile (as found on Debian systems)
AC_CHECK_HEADER([lockfile.h], [AC_CHECK_LIB(lockfile, lockfile_create)])

dnl and whether its companion program dotlockfile is setgid
AC_PATH_PROG(dotlockfilepath, dotlockfile, no)
if test "$ac_cv_lib_lockfile_lockfile_create" != "no" ; then
  if test "$ac_cv_path_dotlockfilepath" != "no" ; then
    AC_CACHE_CHECK(whether dotlockfile is setgid, nmh_cv_dotlockfile_setgid,
    [ if test -g "$ac_cv_path_dotlockfilepath" ; then
        nmh_cv_dotlockfile_setgid=yes
      else
        nmh_cv_dotlockfile_setgid=no
      fi])
  fi
fi

dnl Provide a way for distcheck to disable setgid_mail via
dnl DISTCHECK_CONFIGURE_FLAGS.
AS_IF([test x"$DISABLE_SETGID_MAIL" != x -a x"$DISABLE_SETGID_MAIL" != x0],
  [nmh_cv_dotlockfile_setgid=yes])

dnl If mailspool is not world-writable and dotlockfile is not setgid,
dnl we need to #define MAILGROUP to 1 and make inc setgid.
if test x"$with_locking" = x"dot" -a x"$nmh_cv_mailspool_world_writable" = x"no" -a x"$nmh_cv_dotlockfile_setgid" != x"yes" ; then
  dnl do we really need both of these?
  AC_DEFINE([MAILGROUP],[1],
    [Define to 1 if you need to make `inc' set-group-id because your mail spool is not world writable. There are no guarantees as to the safety of doing this, but this #define will add some extra security checks.])dnl
  SETGID_MAIL=1
fi
AC_SUBST([SETGID_MAIL])dnl

dnl Use ls to see which group owns the mail spool directory.
AC_CACHE_CHECK(what group owns the mail spool, nmh_cv_ls_mail_grp,
[nmh_cv_ls_mail_grp=`$lspath -dL $nmh_cv_ls_grpopt $mailspool|$AWK '{print $4}'`
])
MAIL_SPOOL_GRP=$nmh_cv_ls_mail_grp
AC_SUBST([MAIL_SPOOL_GRP])dnl

NMH_MIMETYPEPROC
NMH_MIMEENCODINGPROC

dnl -------------------
dnl CHECK FOR LIBRARIES
dnl -------------------
dnl Checks for network libraries (nsl, socket)
NMH_CHECK_NETLIBS

dnl Check the locking functions supported and what we should use by default
NMH_LOCKING

dnl Check for iconv
NMH_CHECK_ICONV

dnl Check for tputs() callback argument
NMH_TPUTS_PUTC_ARG

termcap_curses_order="termcap tinfo curses ncurses"
for lib in $termcap_curses_order; do
  AC_CHECK_LIB([${lib}], [setupterm], [TERMLIB="-l$lib"; break])
done
AC_SUBST([TERMLIB])dnl
AS_IF([test "x$TERMLIB" = "x"],
      [AC_MSG_FAILURE([Could not find setupterm in any library.  Is there a
curses or ncurses library or package that you can install?])])

dnl Check for readline support
NMH_READLINE

dnl --------------
dnl CHECK FOR NDBM
dnl --------------

AC_ARG_WITH([ndbm],AS_HELP_STRING([--with-ndbm=ARG],[use -lARG to link with ndbm]),
            [nmh_ndbm=$withval],[nmh_ndbm=autodetect])
AC_ARG_WITH([ndbmheader],AS_HELP_STRING([--with-ndbmheader=ARG],[#include <ARG> to use ndbm]),
            [nmh_ndbmheader=$withval],[nmh_ndbmheader=autodetect])

if test "$nmh_ndbm" = "autodetect"; then
  if test "$nmh_ndbmheader" != "autodetect"; then
    AC_MSG_ERROR([must specify both --with-ndbm and --with-ndbmheader or neither])
  else

    dnl There are at least four implementations of ndbm, and
    dnl several of those can be in different places at the whim
    dnl of the system integrator. A good summary of this mess
    dnl can be found at http://www.unixpapa.com/incnote/dbm.html

    dnl Classic ndbm with no library required (eg NetBSD): try this
    dnl first so we don't accidentally link in a pointless but harmless
    dnl library in one of the later ndbm.h+libfoo tests:
    NMH_CHECK_NDBM(ndbm.h,,,
    dnl Berkeley DBv2 emulating ndbm: header in db.h, e.g., 32-bit Cygwin:
      NMH_CHECK_NDBM(db.h,db,,
    dnl Berkeley DBv1 emulating ndbm:
        NMH_CHECK_NDBM(ndbm.h,db,,
          NMH_CHECK_NDBM(ndbm.h,db1,,
    dnl Classic ndbm:
            NMH_CHECK_NDBM(ndbm.h,ndbm,,
    dnl glibc2.1 systems put db1 in a subdir:
              NMH_CHECK_NDBM(db1/ndbm.h,db1,,
    dnl GNU gdbm emulating ndbm, with header possibly in gdbm/
    dnl and possibly needing gbdm_compat library:
                NMH_CHECK_NDBM(gdbm/ndbm.h,gdbm,,
                  NMH_CHECK_NDBM(gdbm/ndbm.h,gdbm_compat -lgdbm,,
                    NMH_CHECK_NDBM(ndbm.h,gdbm,,
    dnl 64-bit Cygwin:
                      NMH_CHECK_NDBM(ndbm.h,gdbm_compat -lgdbm,,
    dnl On Linux, libgdbm_compat.so loads libgdbm.so as well, so it doesn't
    dnl need to be explicit:
                        NMH_CHECK_NDBM(gdbm-ndbm.h,gdbm_compat)))))))))))
  fi
else
  dnl We don't really need to check that the user-specified values work,
  dnl but it is a convenience to the user to bomb out early rather than
  dnl after configure and half the compile process.
  NMH_CHECK_NDBM([$nmh_ndbmheader],[$nmh_ndbm])
fi

if test "$nmh_ndbm_found" = "no"; then
  AC_MSG_ERROR([could not find a working ndbm library/header combination])
else
  dnl Now export the lib/header to our makefile/config.h:
  if test x"$nmh_ndbmheader" != x; then
    AC_DEFINE_UNQUOTED(NDBM_HEADER, <$nmh_ndbmheader>,
      [Define to the header containing the ndbm API prototypes.])
  fi
  if test x"$nmh_ndbm" != x; then
    NDBM_LIBS="-l$nmh_ndbm"
  else
    NDBM_LIBS=
  fi
  AC_SUBST([NDBM_LIBS])
fi

dnl ------------------
dnl Set RPM build root
dnl ------------------
dnl nmhrpm is used in the final summary, see below.  The default value is
dnl reported there as ./RPM, consistent with the reporting of the default
dnl source code location as ., but its absolute path is used in the Makefile.
AC_ARG_WITH([rpmdir],
  [AS_HELP_STRING([--with-rpmdir=RPMDIR], [RPM build directory @<:@RPM@:>@])])
  AS_IF([test x"$with_rpmdir" = x], [rpmdir='$(abs_srcdir)/RPM'; nmhrpm=./RPM],
          [rpmdir="$with_rpmdir"; eval "nmhrpm=${rpmdir}"])
AC_SUBST([rpmdir])


dnl --------------------
dnl CHECK FOR CYRUS-SASL
dnl --------------------

AS_IF([test x"$with_cyrus_sasl" != x"no"],[
  AC_CHECK_HEADER([sasl/sasl.h], HAVE_SASL_H=1)
  AC_CHECK_LIB([sasl2], [sasl_client_new], [SASLLIB="-lsasl2"])])
AC_SUBST([SASLLIB])

dnl -----------------
dnl Enable SASL?
dnl -----------------
dnl By default (with_cyrus_sasl=''), enable SASL if header and lib are found.
dnl If SASL requested (--with-cyrus-sasl with_cyrus_sasl=yes), error if header or lib not found.
dnl If SASL disabled (--without-cyrus-sasl with_cyrus_sasl=no), don't enable it.
sasl_support=no
CYRUS_SASL=0
AC_SUBST([CYRUS_SASL])
AS_IF([test "x$with_cyrus_sasl" = xyes && test "x$HAVE_SASL_H" = x],
      [AC_MSG_ERROR([SASL requested but sasl.h not found])],
      [test "x$with_cyrus_sasl" = xyes && test "x$SASLLIB" = x],
      [AC_MSG_ERROR([SASL requested but Cyrus SASL library not found])],
      [test "x$with_cyrus_sasl" != xno && test "x$HAVE_SASL_H" = x1 &&
       test "x$SASLLIB" != x],
      [AC_DEFINE([CYRUS_SASL], [1],
        [Define to use the Cyrus SASL library for authentication of POP and SMTP.])
       CYRUS_SASL=1
       sasl_support=yes])

dnl -----------------
dnl CHECK FOR OPENSSL
dnl -----------------

AS_IF([test x"$with_tls" != x"no"],[
  dnl OpenBSD 5 needs the other-libraries (fifth argument) to the
  dnl AC_CHECK_LIB for SSL_new, because it doesn't
  dnl automatically append -lcrypto when linking with -lssl.
  AC_CHECK_HEADER([openssl/ssl.h], HAVE_SSL_H=1)
  AC_CHECK_LIB([crypto], [BIO_write], [TLSLIB="-lcrypto"])
  AC_CHECK_LIB([ssl], [SSL_new], [TLSLIB="-lssl $TLSLIB"], [TLSLIB=],
               [$TLSLIB])])
AC_SUBST([TLSLIB])

dnl -----------------
dnl Enable TLS?
dnl -----------------
dnl By default (with_tls=''), enable TLS if header and libs were found.
dnl If TLS requested (--with-tls with_tls=yes), error if header/lib not found.
dnl If TLS disabled (--without-tls with_tls=no), don't enable it.
tls_support=no
TLS_SUPPORT=0
AC_SUBST([TLS_SUPPORT])
AS_IF([test "x$with_tls" = xyes && test "x$HAVE_SSL_H" = x],
      [AC_MSG_ERROR([TLS requested but openssl/ssl.h not found])],
      [test "x$with_tls" = xyes && test "x$TLSLIB" = x],
      [AC_MSG_ERROR([TLS requested but crypto or ssl library not found])],
      [test "x$with_tls" != xno && test "x$HAVE_SSL_H" = x1 && test "x$TLSLIB" != x],
      [AC_DEFINE([TLS_SUPPORT], [1], [Support TLS for session encryption.])
       TLS_SUPPORT=1
       tls_support=yes
       save_LIBS="$LIBS"
       LIBS="$LIBS $TLSLIB"
       AC_CHECK_FUNCS([X509_VERIFY_PARAM_set1_host])
       LIBS="$save_LIBS"])

dnl -----------------
dnl CHECK FOR CURL
dnl -----------------
dnl Look for curl if oauth not disabled (--without-oauth).
AC_PATH_PROG([curl_config], [curl-config])
AS_IF([test "x$with_oauth" != xno && test -n "$curl_config"], [
    save_CFLAGS="$CFLAGS"
    CFLAGS="$AM_CPPFLAGS $AM_CFLAGS $CFLAGS `$curl_config --cflags`"
    AC_CHECK_HEADER([curl/curl.h], [
        HAVE_CURL_H=1
        AC_CHECK_LIB([curl], [curl_easy_init], [
            CURLLIB="`$curl_config --libs`"
            AC_SUBST([CURLLIB])
            CURL_USER_AGENT="`$curl_config --version | sed 's|^libcurl  *|libcurl/|; q'`"
            AS_IF([test "x$CURL_USER_AGENT" != "x`echo $CURL_USER_AGENT | sed 's/ //'`"],
                  [AC_MSG_WARN([unexpected curl-config --version: $CURL_USER_AGENT])
                   CURL_USER_AGENT=libcurl/UNKNOWN])
            AC_SUBST([CURL_USER_AGENT])
          ])
      ])
    CFLAGS="$save_CFLAGS"
  ])

dnl -----------------
dnl Enable OAuth?
dnl -----------------
dnl By default (with_oauth=''), enable OAuth if curl is found.
dnl If OAuth requested (--with-oauth with_oauth=yes), error if curl not found.
dnl If OAuth disabled (--without-oauth with_oauth=no), don't enable it.
oauth_support=no
OAUTH_SUPPORT=0
AC_SUBST([OAUTH_SUPPORT])
AS_IF([test "x$with_oauth" = xyes && test "x$HAVE_CURL_H" = x],
      [AC_MSG_ERROR([OAuth requested but curl/curl.h not found])],
      [test "x$with_oauth" = xyes && test "x$CURLLIB" = x],
      [AC_MSG_ERROR([OAuth requested but curl library not found])],
      [test "x$with_oauth" != xno && test "x$HAVE_CURL_H" = x1 && test "x$CURLLIB" != x],
      [AC_DEFINE([OAUTH_SUPPORT], [1], [Support OAuth2 in SMTP auth.])
       OAUTH_SUPPORT=1
       oauth_support=yes])

dnl ----------------
dnl CHECK FLEX FIXUP
dnl ----------------
dnl Use AM_LFLAGS make variable setting to work around bugs in flex
dnl 2.5.36-37 that cause signed/unsigned mismatch,
dnl http://sourceforge.net/p/flex/bugs/140/
dnl 2.6.1 bug: https://github.com/westes/flex/issues/97
AS_IF([test "$LEX" = flex],
      [AS_CASE([`$LEX -V`],
               [flex\ 2.5.35], [AM_LFLAGS=\
'; sed "s/	int n;/	size_t n;/" $@ >$@.tmp && mv -f $@.tmp $@;'],
               [flex\ 2.5.3[[67]]], [AM_LFLAGS=\
'; sed "s/\(	\)int i;/\1yy_size_t i;/" $@ >$@.tmp && mv -f $@.tmp $@;'],
               [flex\ 2.6.0], [AM_LFLAGS=\
'; sed "s/, num_to_read/, (size_t) num_to_read/" $@ >$@.tmp && mv -f $@.tmp $@;'],
               [flex\ 2.6.1], [AM_LFLAGS=\
'; sed '\''/\<int num_to_read\>/s/int/size_t/; \
                s/\<yy_size_t i\>/int i/; \
                s/int n; \\/yy_size_t n; \\/'\'' $@ >$@.tmp && mv -f $@.tmp $@;'])
       AC_SUBST([AM_LFLAGS])])

dnl ------------------
dnl By default we put backend executables in $(libexecdir)/nmh and
dnl configuration files in $(sysconfdir)/nmh, but let a user override
dnl these choices by supplying these variables
dnl ------------------

AC_ARG_VAR([nmhlibexecdir], [The installation directory for backend programs
used by nmh.  Defaults to $(libexecdir)/nmh.])
AS_IF([test -z ${nmhlibexecdir+x}], [nmhlibexecdir='${libexecdir}/nmh'])

AC_ARG_VAR([nmhetcdir], [The installation directory for configuration files
used by nmh.  Defaults to $(sysconfdir)/nmh.])
AS_IF([test -z ${nmhetcdir+x}], [nmhetcdir='${sysconfdir}/nmh'])

dnl ------------------
dnl FOR INTERNAL USE by the NMH test suite
dnl ------------------
AC_ARG_VAR([NMHETCDIRINST], [for internal use by nmh test suite])
AS_IF([test -n "$NMHETCDIRINST"], [nmhetcdirinst=$NMHETCDIRINST]
      AC_SUBST([nmhetcdirinst]))

dnl ----------------
dnl CHECK STRUCTURES
dnl ----------------

dnl For platforms such as FreeBSD that have tm_gmtoff in struct tm.
dnl (FreeBSD has a timezone() function but not a timezone global
dnl variable that is visible).
dnl On Linux, $AM_CPPFLAGS contains -D_GNU_SOURCE, which makes
dnl tm_gmtoff visible.
nmh_saved_CPPFLAGS="$CPPFLAGS"
CPPFLAGS="$AM_CPPFLAGS $CPPFLAGS"
AC_CHECK_MEMBERS([struct tm.tm_gmtoff],,,[[#include <time.h>]])
CPPFLAGS="$nmh_saved_CPPFLAGS"

AC_STRUCT_DIRENT_D_TYPE

dnl
dnl Sigh, this is required because under the new world order autoconf has
dnl nothing to create in a few of the build directories when doing an object
dnl tree build.  So make sure we created certain directories if they don't
dnl exist.
dnl

AC_CONFIG_COMMANDS([build-directories],
[AS_MKDIR_P([etc]); AS_MKDIR_P([man])])

AC_CONFIG_COMMANDS_POST([

dnl These odd looking assignments are done to expand out unexpanded
dnl variables in bindir et al (for instance mandir is '${datarootdir}/man'),
dnl but expanding that gives '${prefix}/share/man', so we need to expand
dnl again to get the final answer.
dnl We only use the expanded versions to print the install paths in
dnl the final summary and should use them nowhere else (see the autoconf
dnl docs for the rationale for bindir etc being unexpanded).
eval "nmhbin=${bindir}";         eval "nmhbin=${nmhbin}"
eval "nmhlibexec=${nmhlibexecdir}"; eval "nmhlibexec=${nmhlibexec}"
eval "nmhlibexec=${nmhlibexec}"
eval "nmhsysconf=${nmhetcdir}"; eval "nmhsysconf=${nmhsysconf}"
eval "nmhman=${mandir}";         eval "nmhman=${nmhman}"
eval "nmhdoc=${docdir}";         eval "nmhdoc=${nmhdoc}"
eval "nmhrpm=${nmhrpm}";

AC_MSG_NOTICE([
nmh configuration
-----------------
nmh version                : AC_PACKAGE_VERSION
host os                    : ${host}
compiler                   : ${CC}
compiler flags             : ${AM_CFLAGS} ${CFLAGS}
linker flags               : ${AM_LDFLAGS} ${LDFLAGS}
preprocessor flags         : ${AM_CPPFLAGS} ${CPPFLAGS}
source code location       : ${srcdir}
binary install path        : ${nmhbin}
libexec install path       : ${nmhlibexec}
config files install path  : ${nmhsysconf}
man page install path      : ${nmhman}
docs install path          : ${nmhdoc}
RPM build root             : ${nmhrpm}
backup prefix              : ${backup_prefix}
transport system           : ${MTS}
spool default locking type : ${with_locking}
default smtp server        : ${smtpserver}
SASL support               : ${sasl_support}
TLS support                : ${tls_support}
OAuth support              : ${oauth_support}
])])dnl

dnl ---------------
dnl OUTPUT MAKEFILE
dnl ---------------
AC_CONFIG_FILES([Makefile test/common.sh])
AC_CONFIG_FILES([uip/spost], [chmod +x uip/spost])
AC_OUTPUT
