if not cc.has_function('dlsym')
    libdl = cc.find_library('dl')
else
    libdl = []
endif

add_project_arguments('-Wall', language : 'c')
add_project_arguments('-Werror', language : 'c')
add_project_arguments('-fvisibility=hidden', language : 'c')
add_project_arguments('-DWL_HIDE_DEPRECATED', language : 'c')
add_project_arguments('-D_GNU_SOURCE', language : 'c')
add_project_link_arguments('-Wl,-Bsymbolic', language : 'c')

if cc.has_argument('-Wpedantic')
        add_project_arguments('-Wno-pedantic', language : 'c')
endif

wl_protos = dependency('wayland-protocols', version: '>= 1.8')
libdrm = dependency('libdrm')
wl_protos_dir = wl_protos.get_pkgconfig_variable('pkgdatadir')
wl_dmabuf_xml = join_paths(wl_protos_dir, 'unstable', 'linux-dmabuf', 'linux-dmabuf-unstable-v1.xml')
wp_presentation_time_xml = join_paths(wl_protos_dir, 'stable', 'presentation-time', 'presentation-time.xml')
wl_drm_syncobj_xml = join_paths(wl_protos_dir, 'staging', 'linux-drm-syncobj', 'linux-drm-syncobj-v1.xml')

client_header = generator(prog_scanner,
    output : '@BASENAME@-client-protocol.h',
    arguments : ['client-header', '@INPUT@', '@OUTPUT@']
)

if wl_scanner.version().version_compare('>= 1.14.91')
    code_arg = 'private-code'
else
    code_arg = 'code'
endif

code = generator(prog_scanner,
    output : '@BASENAME@-protocol.c',
    arguments : [code_arg, '@INPUT@', '@OUTPUT@']
)

src = [
    'wayland-thread.c',
    'wayland-egldevice.c',
    'wayland-egldisplay.c',
    'wayland-eglstream.c',
    'wayland-eglstream-server.c',
    'wayland-eglsurface.c',
    'wayland-eglswap.c',
    'wayland-eglutils.c',
    'wayland-eglhandle.c',
    'wayland-external-exports.c',
    'wayland-drm.c',

    wayland_eglstream_protocol_c,
    wayland_eglstream_client_protocol_h,
    wayland_eglstream_server_protocol_h,
    wayland_eglstream_controller_protocol_c,
    wayland_eglstream_controller_client_protocol_h,
    wayland_drm_protocol_c,
    wayland_drm_client_protocol_h,
    wayland_drm_server_protocol_h,
]

src += client_header.process(wl_dmabuf_xml)
src += code.process(wl_dmabuf_xml)

src += client_header.process(wp_presentation_time_xml)
src += code.process(wp_presentation_time_xml)

src += client_header.process(wl_drm_syncobj_xml)
src += code.process(wl_drm_syncobj_xml)

egl_wayland = library('nvidia-egl-wayland',
    src,
    dependencies : [
        egl_headers,
        eglexternalplatform,
        wayland_server,
        wayland_client,
        wayland_egl_backend,
        threads,
        libdrm,
        libdl,
    ],
    include_directories : inc,
    version : meson.project_version(),
    install : true,
)

install_data('10_nvidia_wayland.json',
    install_dir: '@0@/egl/egl_external_platform.d'.format(get_option('datadir')))
