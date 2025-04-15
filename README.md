# README.md
这个项目演示了如何在vscode中配置c/c++ gcc编译环境，它解决了3个问题：
- C/C++ 插件的debug功能（解决与clangd冲突，但是正确提供debug的launch）
- clangd 的代码提示和正确索引（如何配置clangd参数）
- meson 与vscode的配合（提供compile_commands.json）文件

用的软件有：
- vscode + meson/clangd/c/c++插件
- meson/ninja 安装在windows中
- clang/clangd/gcc 安装在msys2中

## 环境配置
windows + msys2 + meson + ninja + gcc
```bash
# 安装gcc
pacman -S mingw-w64-ucrt-x86_64-gcc mingw-w64-ucrt-x86_64-gdb mingw-w64-ucrt-x86_64-git 
# 【可选】安装meson和ninja
# 建议直接在windows上安装
pacman -S mingw-w64-ucrt-x86_64-meson mingw-w64-ucrt-x86_64-ninja 
# 安装clang 和 clangd
pacman -S mingw-w64-ucrt-x86_64-clang mingw-w64-ucrt-x86_64-clang-tools-extra 
```

**为什么要在msys2中安装clangd？** 
> 因为vscode中的clangd不能索引msys2中的头文件，所以需要在msys2中安装clangd。才可以使用mingw-w64对应的头文件.
> Windows的头文件和Linux的头文件是不一样的。

然后在vscode中安装clangd，c/c++, meson 插件。  
然后配置 clangd和c++ intellisense。  

- settings.json 禁用c/c++ intellisense，开启和配置clangd
```json
{
    "C_Cpp.default.configurationProvider": "mesonbuild.mesonbuild",
    "C_Cpp.intelliSenseEngine": "disabled", 
    "clangd.enable": true,
    // 使用msys2中的clangd，否则找不到msys2中的头文件
    "clangd.path": "C:/MyTools/scoop_root/apps/msys2/2025-02-21/ucrt64/bin/clangd.exe",
    "clangd.arguments": [
        // 指定 compile_commands.json 文件的路径
        "--compile-commands-dir=${workspaceFolder}/build",
        "--background-index",
        "--completion-style=detailed",
        "--header-insertion=iwyu",
        "--header-insertion-decorators",
        // "--clang-tidy",
        // "--clang-tidy-checks=-*,modernize-*",
        "--pch-storage=memory",
        // 默认格式化风格: 谷歌开源项目代码指南（可用的有 LLVM, Google, Chromium, Mozilla, Webkit, Microsoft, GNU 等）
        "--fallback-style=Microsoft",
        // 指定从那个目录搜索标准库头文件
        "--query-driver=C:/MyTools/scoop_root/apps/msys2/current/ucrt64/include,C:/MyTools/scoop_root/apps/msys2/current/usr/include,C:/MyTools/scoop_root/apps/msys2/current/ucrt64/lib/gcc/x86_64-pc-mingw64/14.2.0/include",
    ],
    "clangd.fallbackFlags": [
        "-std=c++11",
        // "--target=x86_64-pc-windows-msvc",
        "--target=x86_64-pc-mingw64",
        "-I${workspaceFolder}/",
        "-I${workspaceFolder}/include",
        "-I${workspaceFolder}/src",
    ]
}
```
- launch.json 配置编译命令, 手动实现setup和build命令. 配置gdb调试.
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "meson-setup",
            "type": "PowerShell",
            "request": "launch",
            "script": "$env:PATH+=';C:/MyTools/scoop_root/apps/msys2/current/usr/bin;C:/MyTools/scoop_root/apps/msys2/current/ucrt64/bin;'; meson setup build --buildtype=debug --cross-file=mingw-win32.ini",
            "cwd": "${workspaceFolder}",
        },
        {
            "name": "meson-build",
            "type": "PowerShell",
            "request": "launch",
            "script": "meson compile -C build",
            "cwd": "${workspaceFolder}",
        },
        {
            "name": "debug-gdb",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/${config:mesonbuild.buildFolder}/demo.exe",
            "args": ["./res/test1.mp3"],
            "stopAtEntry": true,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "envFile": "${workspaceFolder}/${config:mesonbuild.buildFolder}/meson-vscode.env",
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "C:/MyTools/scoop_root/apps/msys2/current/ucrt64/bin/gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            // "preLaunchTask": "Meson: Build all targets"
        }
    ]
}
```

## 手动指令参考
```bash
# 添加gcc PATH
$env:PATH += ';C:/MyTools/scoop_root/apps/msys2/current/usr/bin;C:/MyTools/scoop_root/apps/msys2/current/ucrt64/bin;'

# setup
meson setup --buildtype=release --cross-file=mingw-win32.ini build

# build
meson compile -C build

# test
meson test -C build

# install
meson install -C build

# cat
file ./out/demo.exe
file ./out/libffaudio-1.dll
```