# SPDX-License-Identifier: CC0-1.0

if meson.is_cross_build()
    py3_dep = dependency('python3', required : get_option('pycamera'))

    if not py3_dep.found()
        pycamera_enabled = false
        subdir_done()
    endif
else
    py = import('python').find_installation('python3', required : get_option('pycamera'))

    if not py.found()
        pycamera_enabled = false
        subdir_done()
    else
        py3_dep = py.dependency(required : get_option('pycamera'))
        if not py3_dep.found()
            pycamera_enabled = false
            subdir_done()
        endif
    endif
endif

pybind11_dep = dependency('pybind11', required : get_option('pycamera'))

if not pybind11_dep.found()
    pycamera_enabled = false
    subdir_done()
endif

pycamera_enabled = true

pycamera_sources = files([
    'py_camera_manager.cpp',
    'py_color_space.cpp',
    'py_enums.cpp',
    'py_geometry.cpp',
    'py_helpers.cpp',
    'py_main.cpp',
    'py_transform.cpp',
])

# Generate controls

gen_py_controls_input_files = files([
    '../../libcamera/control_ids.yaml',
    'py_controls_generated.cpp.in',
])

gen_py_controls = files('gen-py-controls.py')

pycamera_sources += custom_target('py_gen_controls',
                                  input : gen_py_controls_input_files,
                                  output : ['py_controls_generated.cpp'],
                                  command : [gen_py_controls, '--mode', 'controls', '-o', '@OUTPUT@', '@INPUT@'])

# Generate properties

gen_py_property_enums_input_files = files([
    '../../libcamera/property_ids.yaml',
    'py_properties_generated.cpp.in',
])

pycamera_sources += custom_target('py_gen_properties',
                                  input : gen_py_property_enums_input_files,
                                  output : ['py_properties_generated.cpp'],
                                  command : [gen_py_controls, '--mode', 'properties', '-o', '@OUTPUT@', '@INPUT@'])

# Generate formats

gen_py_formats_input_files = files([
    '../../libcamera/formats.yaml',
    'py_formats_generated.cpp.in',
])

gen_py_formats = files('gen-py-formats.py')

pycamera_sources += custom_target('py_gen_formats',
                                  input : gen_py_formats_input_files,
                                  output : ['py_formats_generated.cpp'],
                                  command : [gen_py_formats, '-o', '@OUTPUT@', '@INPUT@'])

pycamera_deps = [
    libcamera_private,
    py3_dep,
    pybind11_dep,
]

pycamera_args = [
    '-fvisibility=hidden',
    '-Wno-shadow',
    '-DPYBIND11_USE_SMART_HOLDER_AS_DEFAULT',
]

if meson.is_cross_build()
    destdir = get_option('libdir') / ('python' + py3_dep.version()) / 'site-packages' / 'libcamera'

    pycamera = shared_module('_libcamera',
                             pycamera_sources,
                             install : true,
                             install_dir : destdir,
                             name_prefix : '',
                             dependencies : pycamera_deps,
                             cpp_args : pycamera_args)
else
    pycamera = py.extension_module('_libcamera',
                                   pycamera_sources,
                                   install : true,
                                   subdir : 'libcamera',
                                   dependencies : pycamera_deps,
                                   cpp_args : pycamera_args)
endif

# Create symlinks from the build dir to the source dir so that we can use the
# Python module directly from the build dir.

run_command('ln', '-fsrT', files('__init__.py'),
            meson.current_build_dir() / '__init__.py',
            check : true)

run_command('ln', '-fsrT', meson.current_source_dir() / 'utils',
            meson.current_build_dir() / 'utils',
            check : true)

if meson.is_cross_build()
    install_data(['__init__.py'], install_dir : destdir)
else
    py.install_sources(['__init__.py'],
                       subdir : 'libcamera',
                       pure : false)
endif

# \todo Generate stubs when building. See https://peps.python.org/pep-0484/#stub-files
# Note: Depends on pybind11-stubgen. To generate pylibcamera stubs:
# $ PYTHONPATH=build/src/py pybind11-stubgen --no-setup-py -o build/src/py libcamera
