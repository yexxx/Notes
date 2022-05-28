# 使用vscode进行Debug

0. 提前写好`CMakeLists.txt`文件并运行，安装`cmake-tools`插件，在保存`CMakeLists.txt`文件时该插件会完成所有配置。

1. `task.json`

    ```json
    {
        "tasks": [
            {
                "label": "build",
                "command": "make",
                "args": [],
                "type": "shell",
                "options": {
                    "cwd": "${workspaceFolder}/build"
                }
            },
        ],
        "version": "2.0.0"
    }
    ```

2. `launch.json`

    ```json
    {
    "configurations": [
        {
            "name": "debug",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description": "Set Disassembly Flavor to Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "build"
        }
    ]
    }
    ```

3. 预设变量参考
    `${workspaceFolder}` - 在VS Code中打开的文件夹的路径
    `${workspaceRoot}` - VS Code当前打开的文件夹（不推荐使用该变量，多工作区不方便使用）
    `${workspaceFolderBasename}` - VS代码中打开的文件夹的名称，没有任何斜杠（/）
    `${file}` - 当前打开的文件
    `${relativeFile}` - 当前打开的文件相对于workspaceFolder
    `${fileBasename}` - 当前打开文件的基本名称
    `${fileBasenameNoExtension}` - 当前打开文件的基本名称，没有文件扩展名
    `${fileDirname}` - 当前打开文件的目录名
    `${fileExtname}` - 当前打开文件的扩展名
    `${cwd}` - 任务运行器在启动时的当前工作目录
    `${lineNumber}` - 活动文件中当前选定的行号
    `${selectedText}` - 活动文件中当前选定的文本
