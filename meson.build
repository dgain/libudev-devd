project('libudev-devd', 'c',
	version : '0.4.1',
	license : 'BSD2CLAUSE',
	default_options : [
		'buildtype=debugoptimized',
		'warning_level=1',
		'c_std=c11',
		'werror=true' ],
	meson_version : '>=0.50.0')

libudevdevd_version = meson.project_version().split('.')

dir_data        = join_paths(get_option('prefix'), get_option('datadir'))
dir_sysconf     = join_paths(get_option('prefix'), get_option('sysconfdir'))
dir_libexec     = join_paths(get_option('prefix'), get_option('libexecdir'))
dir_lib         = join_paths(get_option('prefix'), get_option('libdir'))
dir_src         = join_paths(meson.source_root())

# Compiler setup
cc = meson.get_compiler('c')
cflags = ['-fvisibility=hidden']
add_project_arguments(cflags, language: 'c')

# config.h
config_h = configuration_data()
config_h_inc = include_directories('.')
if get_option('buildtype') == 'debug' or get_option('buildtype') == 'debugoptimized'
	config_h.set_quoted('MESON_BUILD_ROOT', meson.build_root())
else
	config_h.set_quoted('MESON_BUILD_ROOT', '')
endif

if cc.has_header('devinfo.h')
	devinfo_dep = cc.find_library('devinfo')
	config_h.set('HAVE_DEVINFO_H', '1')
endif

procstat_inc = '''#include <sys/param.h>
#include <sys/queue.h>
#include <sys/socket.h>
'''
if cc.has_header_symbol('libprocstat.h', 'procstat_open_sysctl', prefix : procstat_inc)
	procstat_dep = cc.find_library('procstat')
	config_h.set('HAVE_LIBPROCSTAT_H', '1')
endif

if cc.has_header('linux/input.h')
	config_h.set('HAVE_LINUX_INPUT_H', '1')
endif

if cc.has_function('pipe2')
	config_h.set('HAVE_PIPE2', '1')
endif

if cc.has_function('strchrnul')
	config_h.set('HAVE_STRCHRNUL', '1')
endif

libudevdevd_so_version = '0.0.0'
# Dependencies
thread_dep = dependency('threads')

pkgconfig = import('pkgconfig')

install_headers('libudev.h')
src_libudevdevd = [ 'udev.c',
	'udev-device.c',
	'udev-device.h',
	'udev-enumerate.c',
	'udev-filter.c',
	'udev-filter.h',
	'udev-list.c',
	'udev-list.h',
	'udev-monitor.c',
	'udev-utils.c',
	'udev-utils.h',
	'utils.c',
	'utils.h'
]

deps_libudevdevd = [
	thread_dep,
	devinfo_dep,
	procstat_dep
]

lib_libudevdevd = shared_library('udev',
	src_libudevdevd,
	include_directories : config_h_inc,
	dependencies : deps_libudevdevd,
	version : libudevdevd_so_version,
	install : true
)

pkgconfig.generate(lib_libudevdevd,
	name : 'libudev',
	url : 'https://github.com/FreeBSDDesktop/libudev-devd',
	description : 'Library to access udev device information',
	version : '199', # XXX - should be a proper version
)

# output files
configure_file(output : 'config.h', install : false, configuration : config_h)
