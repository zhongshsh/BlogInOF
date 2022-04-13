# VSCode debug 工具

工作目录：VSCode 远程打开的文件夹

## debug 

- VSCode 安装插件 Native Debug 和 C++

- 在服务器工作目录下新建 .vscode/launch.json，文件内容为

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "GDB",
            "type": "gdb",
            "request": "launch",
            "target": "/home/zhongshanshan/anaconda3/bin/python",       // python 环境
            "arguments": "/home/zhongshanshan/workspace/test/debug.py", // debug 脚本
            "cwd": "/home/zhongshanshan/workspace/log",                 // 日志所在位置
            "valuesFormatting": "parseText",
            "env": { //调 oneflow python 代码要加这个
                "PYTHONPATH": "/home/zhongshanshan/workspace/f/python"  // oneflow 路径
            },
        }
    ]
}
```

- 此时在运行文件按 F5 即开始调试。可以在 python 或者 C++ 加断点进行调试，VSCode 会输出调用栈等信息，添加断点要注意是否在 Setting 中设置允许添加断点

![image](https://user-images.githubusercontent.com/62104945/163183530-8f17dbf5-44cf-4bf8-8eb5-5c96fe7477ff.png)



## 点击跳转

- 切换到 oneflow build 目录，在之前运行Cmake 的 flag 的基础上添加一个 cmake 参数 `-DCMAKE_EXPORT_COMPILE_COMMANDS=1` 运行 cmake。
- 在 oneflow 文件夹下创建软链接 `ln -s build/compile_commands.json ../compile_commands.json`，将 compile_commands.json 链接到工作目录
- 下载并解压 [LLVM](https://github.com/clangd/clangd/releases)
- 在服务器工作目录下新建 .vscode/settings.json，文件内容为

```json
{
    "clangd.path": "/home/zhongshanshan/clangd_14.0.0/bin/clangd",  // LLWM 路径
    "clangd.arguments": [
        "-j",
        "48",
        "-clang-tidy"
    ]
}
```

- 重新连接服务器
