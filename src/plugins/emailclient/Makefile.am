plugindir = $(libdir)/caja-sendto/plugins

AM_CPPFLAGS =                                      \
        -DDATADIR=\"$(datadir)\"                \
        -DICONDIR=\"$(icondir)\"                \
        -DLOCALEDIR="\"$(datadir)/locale\""     \
        -I$(top_srcdir)/src                     \
        -I$(top_builddir)                       \
        $(CAJA_SENDTO_CFLAGS)                   \
        $(EMAILCLIENT_CFLAGS)                   \
        $(DISABLE_DEPRECATED)                   \
        $(WARN_CFLAGS)

plugin_LTLIBRARIES = libnstemailclient.la

libnstemailclient_la_SOURCES = emailclient.c
libnstemailclient_la_LDFLAGS = -module -avoid-version
libnstemailclient_la_LIBADD = $(CAJA_SENDTO_LIBS) $(EMAILCLIENT_LIBS)
