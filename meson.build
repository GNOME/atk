project('atk', 'c',
        version: '2.24.1',
        license: 'LGPLv2.1+',
        default_options: [
          'buildtype=debugoptimized',
          'warning_level=1',
          'c_std=c99',
        ],
        meson_version : '>= 0.40.1')

cc = meson.get_compiler('c')
host_system = host_machine.system()

version = meson.project_version().split('.')
atk_major_version = version[0].to_int()
atk_minor_version = version[1].to_int()
atk_micro_version = version[2].to_int()

atk_interface_age = 1
atk_binary_age = 10000 * atk_major_version + 100 * atk_minor_version + 10 + atk_micro_version

atk_api_version = '1.0'.format(atk_major_version)
atk_api_path = 'atk-@0@/atk'.format(atk_api_version)

atk_prefix = get_option('prefix')
atk_libdir = join_paths(atk_prefix, get_option('libdir'))
atk_sysconfdir = join_paths(atk_prefix, get_option('sysconfdir'))
atk_includedir = join_paths(atk_prefix, get_option('includedir'))
atk_datadir = join_paths(atk_prefix, get_option('datadir'))
atk_libexecdir = join_paths(atk_prefix, get_option('libexecdir'))

atk_conf = configuration_data()

atk_conf.set_quoted('VERSION', meson.project_version())
atk_conf.set_quoted('GETTEXT_PACKAGE', meson.project_name())

# Maintain version scheme with libtool
atk_soversion = '0.@0@.@1@'.format((atk_binary_age - atk_interface_age), atk_interface_age)

add_project_arguments([ '-DG_DISABLE_SINGLE_INCLUDES', '-DATK_DISABLE_SINGLE_INCLUDES' ], language: 'c')

# Compiler and linker flags
common_cflags = []
common_ldflags = []

test_cflags = []

# Symbol visibility
if get_option('default_library') != 'static'
  if host_system == 'windows'
    atk_conf.set('DLL_EXPORT', true)
    atk_conf.set('_ATK_EXTERN', '__declspec(dllexport) extern')
    if cc.get_id() != 'msvc'
      test_cflags += ['-fvisibility=hidden']
    endif
  else
    atk_conf.set('_ATK_EXTERN', '__attribute__((visibility("default"))) extern')
    test_cflags += ['-fvisibility=hidden']
  endif
endif

# Check all compiler flags
foreach cflag: test_cflags
  if cc.has_argument(cflag)
    common_cflags += [ cflag ]
  endif
endforeach

# Linker flags
if host_machine.system() == 'linux'
  foreach ldflag: [ '-Wl,-Bsymbolic', '-Wl,-z,relro', '-Wl,-z,now', ]
    if cc.has_argument(ldflag)
      common_ldflags += [ ldflag ]
    endif
  endforeach
endif

# Maintain compatibility with autotools on macOS
if host_machine.system() == 'darwin'
  common_ldflags += [ '-compatibility_version=1', '-current_version=1.0', ]
endif

# Functions
checked_funcs = [
  'bind_textdomain_codeset',
]

foreach f: checked_funcs
  if cc.has_function(f)
    atk_conf.set('HAVE_' + f.underscorify().to_upper(), 1)
  endif
endforeach

# Dependencies
gobject_req_version = '>= 2.31.2'

gobject_dep = dependency('gobject-2.0', version: gobject_req_version)

# Compat variables for pkgconfig
pkgconf = configuration_data()
pkgconf.set('prefix', atk_prefix)
pkgconf.set('exec_prefix', atk_prefix)
pkgconf.set('libdir', atk_libdir)
pkgconf.set('includedir', atk_includedir)
pkgconf.set('VERSION', meson.project_version())
pkgconf.set('GLIB_PACKAGES', 'gobject-2.0')
pkgconf.set('ATK_API_VERSION', atk_api_version)
pkgconf.set('srcdir', '.')

foreach pkg: [ 'atk.pc', ]
  configure_file(input: pkg + '.in',
                 output: pkg,
                 configuration: pkgconf,
                 install: true,
                 install_dir: join_paths(atk_libdir, 'pkgconfig'))
endforeach

gnome = import('gnome')

# Internal configuration header
configure_file(output: 'config.h', configuration: atk_conf)

root_inc = include_directories('.')

subdir('atk')
subdir('tests')
subdir('po')

if get_option('enable_docs')
  subdir('docs')
endif