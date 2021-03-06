AC_PREREQ(2.52)

AC_INIT(caja-sendto, 1.6.0, http://www.mate-desktop.org)
AC_CONFIG_HEADERS(config.h)

AM_INIT_AUTOMAKE([1.9 dist-bzip2 no-dist-gzip check-news])

# Enable silent build when available (Automake 1.11)
m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])

AM_MAINTAINER_MODE

AM_DISABLE_STATIC
AC_PROG_CC
AM_PROG_LIBTOOL
AC_PROG_INSTALL
AC_PROG_MAKE_SET
GLIB_GSETTINGS

dnl gtk-doc checks
GTK_DOC_CHECK(1.9)

MATE_COMMON_INIT
MATE_DEBUG_CHECK
MATE_COMPILE_WARNINGS([maximum])
MATE_CXX_WARNINGS
MATE_MAINTAINER_MODE_DEFINES

ERROR_CFLAGS="-Wall -Wextra -Werror -Wno_missing_field_initializers -Wno_unused_parameter"

dnl --------------------------------------------------

GLIB_REQUIRED=2.25.9
GTHREAD_REQUIRED=2.6.0
GTK_REQUIRED=2.18
CAJA_EXTENSION=1.1.0
DBUS_REQUIRED=1.0
DBUS_GLIB_REQUIRED=0.60
GUPNP_REQUIRED=0.13

AC_SUBST(GLIB_REQUIRED)
AC_SUBST(GTHREAD_REQUIRED)
AC_SUBST(GTK_REQUIRED)
AC_SUBST(CAJA_EXTENSION)
AC_SUBST(BLUETOOTH_REQUIRED)
AC_SUBST(DBUS_REQUIRED)
AC_SUBST(GUPNP_REQUIRED)

PKG_CHECK_MODULES(CAJA_SENDTO,\
	 glib-2.0 >= $GLIB_REQUIRED                \
	 gthread-2.0 >= $GTHREAD_REQUIRED         \
	 gmodule-2.0 >= $GLIB_REQUIRED		  \
	 gtk+-2.0    >= $GTK_REQUIRED)
AC_SUBST(CAJA_SENDTO_CFLAGS)
AC_SUBST(CAJA_SENDTO_LIBS)

PKG_CHECK_MODULES(CAJA_EXT_SENDTO,\
	 libcaja-extension >= $CAJA_EXTENSION)
AC_SUBST(CAJA_EXT_SENDTO_CFLAGS)
AC_SUBST(CAJA_EXT_SENDTO_LIBS)

dnl Get caja extensions directory
dnl ---------------------------------

AC_ARG_WITH(cajadir,
	    AS_HELP_STRING([--with-cajadir=DIR],[Installation path for Caja extension @<:@auto@:>@]),
	    [ac_with_cajadir=$withval],
	    [ac_with_cajadir=""])
if test "${ac_with_cajadir}" = ""; then
	ac_with_cajadir=`pkg-config --variable=extensiondir libcaja-extension`
fi

AC_MSG_NOTICE([installing caja plugin in ${ac_with_cajadir}])
AC_SUBST([CAJA_EXTENSION_DIR],[${ac_with_cajadir}])

# The full list of plugins
allowed_plugins="emailclient gajim caja-burn pidgin removable-devices upnp"

plugin_error_or_ignore()
{
	if test "${error_on_bad_plugin}" = "1" ; then
		AC_MSG_ERROR([$1])
	else
		AC_MSG_WARN([$1 (disabling plugin)])
	fi
}

AC_ARG_WITH([plugins],
	[AS_HELP_STRING([--with-plugins],[Which caja-sendto plugins to compile (default: auto; "all", "autodetect" are valid)])],
	[], [with_plugins=autodetect])

used_plugins=""
used_plugins2=""

if test "x${with_plugins}" = "xautodetect" ; then
	# Default to all plugins and just ignore them if they won't compile
	with_plugins="all"
	error_on_bad_plugin="0"
else
	# Clean up the plugin list and error if a plugin won't compile
	with_plugins=`echo ,"${with_plugins}", | sed -e 's/[[	,]][[	,]]*/ /g' -e 's/,$//'`
	error_on_bad_plugin="1"
fi

# Iterate through the wanted plugins and weed out bad ones (or error)
for wanted_plugin in ${with_plugins}; do
	for allowed_plugin in ${allowed_plugins}; do
		case ${wanted_plugin} in
			all)
				# List all plugins as being used
				used_plugins=${allowed_plugins}
			;;
			autodetect)
				# List all plugins as being used
				used_plugins=${allowed_plugins}
			;;
			${allowed_plugin}) used_plugins="${used_plugins} ${wanted_plugin}";; # List the plugin
			*) ;;
		esac
	done
done

# Check for plugin-specific requirements and error if necessary
for plugin in ${used_plugins}; do
	add_plugin="1"

	case ${plugin} in
		emailclient)
			PKG_CHECK_MODULES(EMAILCLIENT, gio-2.0,
					  enable_emailclient=yes, enable_emailclient=no)
			if test "${enable_emailclient}" != "yes" ; then
				plugin_error_or_ignore "you need gio-2.0 to build the emailclient plugin"
				add_plugin="0"
			fi
		;;
		gajim)
			AC_ARG_WITH(gajim,
				    [AS_HELP_STRING(--with-gajim,[Path to Gajim install prefix])],
				    [GAJIM_PATH="$withval"],[GAJIM_PATH="$prefix"])

			GAJIM_SHARE_DIR="$GAJIM_PATH"/share/gajim

			PKG_CHECK_MODULES(DBUS, dbus-1 >= $DBUS_REQUIRED dbus-glib-1 >= $DBUS_GLIB_REQUIRED,
					  [
					   AC_SUBST(DBUS_CFLAGS)
					   AC_SUBST(DBUS_LIBS)
					   enable_gajim=yes
					   AC_DEFINE_UNQUOTED(GAJIM_SHARE_DIR, "$GAJIM_SHARE_DIR", [path to gajim share dir])
					   ],
					   [
					    enable_gajim=no
					    ]
					   )
			if test "${enable_gajim}" != "yes" ; then
				plugin_error_or_ignore "you need gajim and dbus-glib installed for the gajim plugin"
				add_plugin="0"
			fi
		;;
		caja-burn)
			PKG_CHECK_MODULES(GIO, gio-2.0,
					  enable_burn=yes, enable_burn=no)
			if test "${enable_burn}" != "yes" ; then
				plugin_error_or_ignore "you need gio installed for the caja-burn plugin"
				add_plugin="0"
			fi
		;;
		pidgin)
			PKG_CHECK_MODULES(PIDGIN, dbus-glib-1 >= $DBUS_GLIB_REQUIRED,
					  enable_pidgin=yes, enable_pidgin=no)
			if test "${enable_pidgin}" != "yes" ; then
				plugin_error_or_ignore "you need dbus-glib to build the pidgin plugin"
				add_plugin="0"
			fi
		;;
		removable-devices)
			PKG_CHECK_MODULES(GIO, gio-2.0,
					  enable_removable=yes, enable_removable=no)
			if test "${enable_removable}" != "yes" ; then
				plugin_error_or_ignore "you need gio-2.0 to build the removable-devices plugin"
				add_plugin="0"
			fi
		;;
		upnp)
			PKG_CHECK_MODULES(UPNP, gupnp-1.0 >= $GUPNP_REQUIRED,
					  enable_upnp=yes, enable_upnp=no)
			if test "${enable_upnp}" != "yes" ; then
				plugin_error_or_ignore "you need gupnp installed for the upnp plugin"
				add_plugin="0"
			fi
		;;
	esac

	# Add the specified plugin
	if test "${add_plugin}" = "1" ; then
		used_plugins2="${used_plugins2} ${plugin}"
	fi
done

ALL_PLUGINS=$allowed_plugins
PLUGINS=$used_plugins2

dnl Error out if no plugins will be built
if test -z "${PLUGINS}" ; then
	AC_MSG_ERROR([No plugins will be built, and caja-sendto requires at least one])
fi

AC_MSG_CHECKING([which plugins to compile])
AC_MSG_RESULT([$PLUGINS])

AC_SUBST([ALL_PLUGINS])
AC_SUBST([PLUGINS])

dnl For the pidgin plugin
AM_CONDITIONAL(HAVE_PIDGIN, test "x$enable_pidgin" = "xyes")

dnl For the GIO based plugins
enable_nst_common=no
if test "x$enable_burn" = "xyes" -o "x$enable_removable" = "xyes"; then
	enable_nst_common=yes
fi
AM_CONDITIONAL(HAVE_GIO, test "x$enable_nst_common" = "xyes")

GLIB_GENMARSHAL=`$PKG_CONFIG --variable=glib_genmarshal glib-2.0`
AC_SUBST(GLIB_GENMARSHAL)

IT_PROG_INTLTOOL([0.35.0])

GETTEXT_PACKAGE=caja-sendto
AC_SUBST(GETTEXT_PACKAGE)
AC_DEFINE_UNQUOTED(GETTEXT_PACKAGE, "$GETTEXT_PACKAGE", [GetText Package])

AM_GLIB_GNU_GETTEXT

AC_OUTPUT([
Makefile
src/Makefile
src/caja-sendto.pc
src/plugins/Makefile
src/plugins/emailclient/Makefile
src/plugins/gajim/Makefile
src/plugins/caja-burn/Makefile
src/plugins/pidgin/Makefile
src/plugins/removable-devices/Makefile
src/plugins/upnp/Makefile
docs/Makefile
docs/caja-sendto/Makefile
docs/caja-sendto/version.xml
po/Makefile.in
])

echo "-------------------------------------------------------"
echo "                                                       "

if test "x${PLUGINS}" != "x" ; then
	for allowed_plugin in ${ALL_PLUGINS}; do
		for plugin in ${PLUGINS}; do
			case ${allowed_plugin} in
				${plugin})
					AC_MSG_NOTICE([** ${allowed_plugin} plugin enabled])
					continue 2
				;;
				*);;
			esac
		done
		AC_MSG_NOTICE([   ${allowed_plugin} plugin disabled])
	done
else
	AC_MSG_NOTICE([   No caja-sendto plugins enabled])
fi

echo "                                                       "
echo "-------------------------------------------------------"
