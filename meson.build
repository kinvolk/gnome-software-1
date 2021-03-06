project('gnome-software', 'c',
  version : '3.24.1',
  default_options : ['warning_level=1'],
  meson_version : '>=0.37.0'
)

conf = configuration_data()
conf.set_quoted('VERSION', meson.project_version())
conf.set_quoted('PACKAGE_NAME', meson.project_name())
conf.set_quoted('PACKAGE_VERSION', meson.project_version())

# this refers to the gnome-software plugin API version
# this is not in any way related to a package or soname version
conf.set_quoted('GS_PLUGIN_API_VERSION', '11')

# install docs
install_data('README', install_dir : 'share/doc/gnome-software')

# get suported warning flags
test_args = [
  '-fstack-protector-strong',
  '-Waggregate-return',
  '-Warray-bounds',
  '-Wcast-align',
  '-Wclobbered',
  '-Wdeclaration-after-statement',
  '-Wempty-body',
  '-Wextra',
  '-Wformat=2',
  '-Wformat-nonliteral',
  '-Wformat-security',
  '-Wformat-signedness',
  '-Wignored-qualifiers',
  '-Wimplicit-function-declaration',
  '-Winit-self',
  '-Winline',
  '-Wmissing-declarations',
  '-Wmissing-format-attribute',
  '-Wmissing-include-dirs',
  '-Wmissing-noreturn',
  '-Wmissing-parameter-type',
  '-Wmissing-prototypes',
  '-Wnested-externs',
  '-Wno-discarded-qualifiers',
  '-Wno-missing-field-initializers',
  '-Wno-strict-aliasing',
  '-Wno-suggest-attribute=format',
  '-Wno-unused-parameter',
  '-Wold-style-definition',
  '-Woverride-init',
  '-Wpacked',
  '-Wpointer-arith',
  '-Wredundant-decls',
  '-Wreturn-type',
  '-Wshadow',
  '-Wsign-compare',
  '-Wstrict-aliasing',
  '-Wstrict-prototypes',
  '-Wswitch-default',
  '-Wtype-limits',
  '-Wundef',
  '-Wuninitialized',
  '-Wunused-but-set-variable',
  '-Wwrite-strings'
]
cc = meson.get_compiler('c')
foreach arg: test_args
  if cc.has_argument(arg)
    add_project_arguments(arg, language : 'c')
  endif
endforeach

# enable full RELRO where possible
# FIXME: until https://github.com/mesonbuild/meson/issues/1140 is fixed
global_link_args = []
test_link_args = [
  '-Wl,-z,relro',
  '-Wl,-z,now',
]
foreach arg: test_link_args
  if cc.has_argument(arg)
    global_link_args += arg
  endif
endforeach
add_global_link_arguments(
  global_link_args,
  language: 'c'
)

appstream_glib = dependency('appstream-glib', version : '>= 0.6.5')
gdk_pixbuf = dependency('gdk-pixbuf-2.0', version : '>= 2.31.5')
gio_unix = dependency('gio-unix-2.0')
gmodule = dependency('gmodule-2.0')
gtk = dependency('gtk+-3.0', version : '>= 3.20.0')
json_glib = dependency('json-glib-1.0', version : '>= 1.1.1')
libm = cc.find_library('libm', required: false)
libsecret = dependency('libsecret-1')
libsoup = dependency('libsoup-2.4', version : '>= 2.51.92')
sqlite3 = dependency('sqlite3')

if get_option('enable-gnome-desktop')
  gnome_desktop = dependency('gnome-desktop-3.0', version : '>= 3.17.92')
endif

if get_option('enable-gtkspell')
  gtkspell = dependency('gtkspell3-3.0')
endif

if get_option('enable-polkit')
  polkit = dependency('polkit-gobject-1')
endif

if get_option('enable-ostree')
  ostree = dependency('ostree-1')
endif

if get_option('enable-packagekit')
  packagekit = dependency('packagekit-glib2', version : '>= 1.1.0')
  conf.set('HAVE_PACKAGEKIT', '1')
  add_project_arguments('-DI_KNOW_THE_PACKAGEKIT_GLIB2_API_IS_SUBJECT_TO_CHANGE',
                        language : 'c')
endif

if get_option('enable-firmware')
  fwupd = dependency('fwupd', version : '>= 0.7.0')
endif

if get_option('enable-flatpak')
  flatpak = dependency('flatpak', version : '>= 0.6.12')
endif

if get_option('enable-limba')
  limba = dependency('limba', version : '>= 0.5.6')
endif

if get_option('enable-rpm')
  rpm = dependency('rpm')
endif

if get_option('enable-ubuntu-reviews')
  oauth = dependency('oauth')
endif

if get_option('enable-gudev')
  gudev = dependency('gudev-1.0')
endif

if get_option('enable-snap')
  snap = dependency('snapd-glib')
endif

gnome = import('gnome')
i18n = import('i18n')

conf.set_quoted('GETTEXT_PACKAGE', meson.project_name())
conf.set_quoted('TESTDATADIR', join_paths(meson.current_source_dir(), 'data'))
conf.set_quoted('LOCALPLUGINDIR',
                join_paths(meson.current_build_dir(),
                           'src', 'plugins'))
conf.set_quoted('LOCALEDIR',
                join_paths(get_option('prefix'),
                           get_option('localedir')))
conf.set_quoted('DATADIR',
                join_paths(get_option('prefix'),
                           get_option('datadir')))
conf.set_quoted('LIBDIR',
                join_paths(get_option('prefix'),
                           get_option('libdir')))
conf.set_quoted('SYSCONFDIR',
                join_paths(get_option('prefix'),
                           get_option('sysconfdir')))
conf.set_quoted('LOCALSTATEDIR',
                join_paths(get_option('prefix'),
                           get_option('localstatedir')))
configure_file(
  output : 'config.h',
  configuration : conf
)
top_build_incdir = include_directories(meson.current_build_dir())

add_project_arguments('-DI_KNOW_THE_GNOME_SOFTWARE_API_IS_SUBJECT_TO_CHANGE',
                      language : 'c')

subdir('src')
subdir('po')
subdir('data')
subdir('doc')

archiver = find_program('git', required : false)
if archiver.found()
  run_target('dist',
    # git config tar.tar.xz.command "xz -c"
    command: [
      'git', 'archive',
      '--prefix=' + meson.project_name() + '-' + meson.project_version() + '/',
      'HEAD',
      '--format=tar.xz',
      '--output',
      meson.project_name() + '-' + meson.project_version() + '.tar.xz'
    ]
  )
else
  message('git not found, you will not be able to run `ninja dist`')
endif

appstream_util = find_program('appstream-util', required : false)
if appstream_util.found()
  custom_target('NEWS',
    output : 'NEWS',
    input : 'data/appdata/org.gnome.Software.appdata.xml.in',
    command : [appstream_util, 'appdata-to-news', '@INPUT@'],
    capture : true
  )
endif

# FIXME: remove when https://github.com/mesonbuild/meson/issues/837 fixed
meson.add_install_script('meson_post_install.sh')
