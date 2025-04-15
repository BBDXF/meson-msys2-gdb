project(
    'ffaudio',
    'c',
    version: '0.1',
    default_options: [
        'warning_level=3',
        # 'b_lto=true',
        # 'optimization=3',
        'c_std=c11',
    ],
)

cc = meson.get_compiler('c')

# 假设FFmpeg安装在 C:\ffmpeg
ffmpeg_lib_dir = 'C:/MyTools/scoop_root/apps/ffmpeg-shared/current'
ffmepg_lib_dir_lib = ffmpeg_lib_dir / 'lib'
ffmepg_lib_dir_include = ffmpeg_lib_dir / 'include'

ffmpeg_libs = [
    cc.find_library('avcodec', dirs: ffmepg_lib_dir_lib),
    cc.find_library('avformat', dirs: ffmepg_lib_dir_lib),
    cc.find_library('avdevice', dirs: ffmepg_lib_dir_lib),
    cc.find_library('avfilter', dirs: ffmepg_lib_dir_lib),
    cc.find_library('avutil', dirs: ffmepg_lib_dir_lib),
    cc.find_library('swresample', dirs: ffmepg_lib_dir_lib),
    cc.find_library('swscale', dirs: ffmepg_lib_dir_lib),
]

ffmpeg_dep = declare_dependency(
    include_directories: include_directories(ffmepg_lib_dir_include),
    dependencies: ffmpeg_libs,
)

# lib 源文件
lib_sources = ['src/ffaudio.c']

# 动态库
lib = both_libraries(
    'ffaudio',
    lib_sources,
    version: '1.0',
    include_directories: include_directories('src'),
    dependencies: [ffmpeg_dep],
    install: true,
    install_dir: meson.project_source_root() / 'out',
)

shared_lib = lib.get_shared_lib()

install_headers(
    'src/ffaudio.h',
    install_dir: meson.project_source_root() / 'out',
)

# demo 源文件
demo_sources = ['demo.c']

# 可执行文件目标
demo = executable(
    'demo',
    demo_sources,
    link_with: lib.get_static_lib(),
    include_directories: include_directories('src'),
    install: true,
    install_dir: meson.project_source_root() / 'out',
)

# 测试目标
test(
    'ffaudio_test',
    demo,
    args: ['./res/test1.mp3'],
    workdir: meson.project_source_root(),
)
