# 将 GCC 与 MinGW 一起使用

在本教程中，您将配置 Visual Studio Code 以使用来自 [mingw-w64](http://mingw-w64.org) 的 GCC C++ 编译器 (g++) 和 GDB 调试器来创建在 Windows 上运行的程序。

配置完 VS Code 后，你将在 VS Code 中编译和调试一个简单的 Hello World 程序。 本教程不会教您有关 GCC、GDB、Mingw-w64 或 C++ 语言的知识。 对于这些主题，网上有很多很好的资源。

如果您有任何问题，请随时在 [VS Code 文档存储库](https://github.com/microsoft/vscode-docs/issues) 中为本教程提交问题。

## 先决条件

要成功完成本教程，您必须执行以下步骤：

1. 安装 [Visual Studio Code](/download).

1. 安装 [VS Code C/C++ 插件](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)。您可以通过在扩展视图 (`kb(workbench.view.extensions)`) 中搜索“c++”来安装 C/C++ 扩展。

    ![C/C++ extension](images/cpp/cpp-extension.png)

1. 通过 [MSYS2](https://www.msys2.org/) 获取最新版本的 Mingw-w64，它提供 GCC、Mingw-w64 和其他有用的 C++ 工具和库的最新本地构建。 您可以从 MSYS2 页面下载最新的安装程序或使用此 [安装程序链接](https://github.com/msys2/msys2-installer/releases/download/2022-01-18/msys2-x86_64-20220118.exe)。

1. 按照[MSYS2网站](https://www.msys2.org/)上的**安装**说明安装Mingw-w64。 当您将安装实际的 Mingw-w64 工具集（“pacman -S --needed base-devel mingw-w64-x86_64-toolchain”）时，请注意运行每个必需的开始菜单和“pacman”命令，尤其是第 7 步。

1. 使用以下步骤将 Mingw-w64 `bin` 文件夹的路径添加到 Windows `PATH` 环境变量：
   1. 在 Windows 搜索栏中，键入“设置”以打开您的 Windows 设置。
   1. 搜索 **Edit environment variables for your account**。
   1. 在 **User variables** 中选择 `Path` 变量，然后选择 **Edit**。
   1. 选择 **New** 并将 Mingw-w64 目标文件夹路径添加到系统路径。 具体路径取决于您安装的 Mingw-w64 版本以及安装位置。 如果您使用上述设置安装 Mingw-w64，则将其添加到路径：`C:\msys64\mingw64\bin`。
   1. 选择 **OK** 以保存更新的 PATH。 您将需要重新打开任何控制台窗口以使新的 PATH 位置可用。

### 检查您的 MinGW 安装

要检查您的 Mingw-w64 工具是否已正确安装且可用，请打开 **new** 命令提示符并键入：

```bash
g++ --version
gdb --version
```

如果您没有看到预期的输出或 `g++` 或 `gdb` 不是可识别的命令，请确保您的 PATH 条目与编译器所在的 Mingw-w64 二进制位置匹配。 如果该 PATH 条目中不存在编译器，请确保按照 [MSYS2 网站](https://www.msys2.org/) 上的说明安装 Mingw-w64。

## 创建 Hello World 程序

在 Windows 命令提示符下，创建一个名为“projects”的空文件夹，您可以在其中放置所有 VS Code 项目。 然后创建一个名为 `helloworld` 的子文件夹，导航到它，并通过输入以下命令在该文件夹中打开 VS Code：

```cmd
mkdir projects
cd projects
mkdir helloworld
cd helloworld
code .
```

代码 。” 命令在当前工作文件夹中打开 VS Code，该文件夹成为您的“工作区”。 通过选择 **Yes, I trust the authors** 接受 [Workspace Trust](/docs/editor/workspace-trust.md) 对话框，因为这是您创建的文件夹。

在学习本教程时，您将看到在工作区的 `.vscode` 文件夹中创建了三个文件：

- `tasks.json` (构建说明)
- `launch.json` (调试器设置)
- `c_cpp_properties.json` (编译器路径和 IntelliSense 设置)

### 添加源代码文件

在文件资源管理器的标题栏中，选择 **New File** 按钮并将文件命名为 `helloworld.cpp`。

![New File title bar button](images/mingw/new-file-button.png)

### 添加hello world源代码

现在粘贴此源代码：

```cpp
#include <iostream>
#include <vector>
#include <string>

using namespace std;

int main()
{
    vector<string> msg {"Hello", "C++", "World", "from", "VS Code", "and the C++ extension!"};

    for (const string& word : msg)
    {
        cout << word << " ";
    }
    cout << endl;
}
```

现在按 `kb(workbench.action.files.save)` 保存文件。 请注意您刚刚添加的文件是如何出现在 VS Code 侧边栏中的 **File Explorer** 视图 (`kb(workbench.view.explorer)`) 中的：

![File Explorer](images/mingw/file-explorer-mingw.png)

您还可以启用 [自动保存](/docs/editor/codebasics.md#saveauto-save) 以自动保存文件更改，方法是在 **File** 主菜单中选中 **Auto Save**。

最左侧的活动栏可让您打开不同的视图，例如 **Search**、**Source Control** 和 **Run**。 您将在本教程后面看到 **Run** 视图。 您可以在 VS Code [用户界面文档](/docs/getstarted/userinterface.md) 中找到有关其他视图的更多信息。

>**注意**：当您保存或打开 C++ 文件时，您可能会看到来自 C/C++ 扩展的通知，告知您有 Insiders 版本的可用性，该版本可让您测试新功能和修复。 您可以通过选择“X”（**清除通知**）来忽略此通知。

## 探索智能感知

在新的 `helloworld.cpp` 文件中，将鼠标悬停在 `vector` 或 `string` 上以查看类型信息。 在声明 `msg` 变量后，开始键入 `msg.`，就像调用成员函数时一样。 您应该会立即看到一个显示所有成员函数的完成列表，以及一个显示“msg”对象类型信息的窗口：

![Statement completion IntelliSense](images/wsl/msg-intellisense.png)

可以按`kbstyle(Tab)`键插入选中的成员； 然后，当您添加左括号时，您将看到有关该函数所需的任何参数的信息。

## 建造 helloworld.cpp

接下来，您将创建一个 `tasks.json` 文件来告诉 VS Code 如何构建（编译）程序。 此任务将调用 g++ 编译器以基于源代码创建可执行文件。

从主菜单中，选择 **Terminal** > **Configure Default Build Task**。 在下拉列表中，将显示一个任务下拉列表，其中列出了 C++ 编译器的各种预定义构建任务。 选择 **g++.exe build active file**，它将构建当前在编辑器中显示（活动）的文件。

![Tasks C++ build dropdown](images/mingw/build-active-file.png)

这将在 `.vscode` 文件夹中创建一个 `tasks.json` 文件并在编辑器中打开它。

您的新 `tasks.json` 文件应类似于以下 JSON：

```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: g++.exe build active file",
            "command": "C:/msys64/mingw64/bin/g++.exe",
            "args": [
                "-g",
                "${file}",
                "-o",
                "${fileDirname}\\${fileBasenameNoExtension}.exe"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "compiler: C:/msys64/mingw64/bin/g++.exe"
        }
    ],
    "version": "2.0.0"
}
```

`command` 设置指定要运行的程序； 在这种情况下是 g++。 `args` 数组指定将传递给 g++ 的命令行参数。 这些参数必须按照编译器预期的顺序指定。 此任务告诉 g++ 获取活动文件 (`${file}`)，编译它，并在当前目录 (`${fileDirname}`) 中创建一个与活动文件同名但带有 ` .exe` 扩展名（`${fileBasenameNoExtension}.exe`），在我们的示例中生成 `helloworld.exe`。

>**注意**：您可以在 [变量参考](/docs/editor/variables-reference.md) 中了解有关 `tasks.json` 变量的更多信息。

`label` 值是您将在任务列表中看到的； 你可以随意命名它。

`group` 对象中的 `"isDefault": true` 值指定该任务将在您按下 `kb(workbench.action.tasks.build)` 时运行。 此属性仅为方便起见； 如果您将其设置为 false，您仍然可以从终端菜单使用 **Tasks: Run Build Task** 运行它。

### 运行构建

1. 回到`helloworld.cpp`。 您的任务构建活动文件并且您想要构建 `helloworld.cpp`。
1. 要运行 `tasks.json` 中定义的构建任务，请按 `kb(workbench.action.tasks.build)` 或从 **Terminal** 主菜单中选择 **Run Build Task**。
1. 当任务开始时，您应该会看到集成终端面板出现在源代码编辑器下方。 任务完成后，终端会显示编译器的输出，指示构建是成功还是失败。 对于成功的 g++ 构建，输出如下所示：

   ![G++ build output in terminal](images/mingw/mingw-build-output.png)

1. 使用 **+** 按钮创建一个新终端，您将拥有一个以 `helloworld` 文件夹作为工作目录的新终端。 运行`dir`，你现在应该看到可执行文件`helloworld.exe`。

    ![Hello World in PowerShell terminal](images/mingw/mingw-dir-output.png)

1. 您可以通过键入 `helloworld.exe`（或 `.\helloworld.exe`，如果您使用 PowerShell 终端）在终端中运行 `helloworld`。

>**注意**：您最初可能需要按几次 `kbstyle(Enter)` 才能在终端中看到 PowerShell 提示符。 此问题应在 Windows 的未来版本中得到修复。

### 修改 tasks.json

您可以修改您的 `tasks.json` 以通过使用类似 `"${workspaceFolder}\\*.cpp"` 的参数而不是 `${file}` 来构建多个 C++ 文件。 这将在当前文件夹中构建所有 `.cpp` 文件。 您还可以通过将 `"${fileDirname}\\${fileBasenameNoExtension}.exe"` 替换为硬编码文件名（例如 `"${workspaceFolder}\\myProgram.exe"`）来修改输出文件名。

## 调试 helloworld.cpp

接下来，您将创建一个 `launch.json` 文件来配置 VS Code 以在您按下 `kb(workbench.action.debug.start)` 来调试程序时启动 GDB 调试器。

1. 从主菜单中，选择 **Run** > **Add Configuration...**，然后选择 **C++ (GDB/LLDB)**。
1. 然后，您将看到各种预定义调试配置的下拉列表。 选择 **g++.exe 构建和调试活动文件**。

![C++ debug configuration dropdown](images/mingw/build-and-debug-active-file.png)

VS Code 创建一个 `launch.json` 文件，在编辑器中打开它，然后构建并运行 'helloworld'。

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "g++.exe - Build and debug active file",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "C:\\msys64\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: g++.exe build active file"
        }
    ]
}
```

`program` 设置指定要调试的程序。 在这里，它被设置为活动文件夹`${fileDirname}` 和活动文件名，扩展名为`.exe` `${fileBasenameNoExtension}.exe`，如果`helloworld.cpp` 是活动文件，则活动文件将是`helloworld.exe`。

默认情况下，C++ 扩展不会向您的源代码添加任何断点，并且 `stopAtEntry` 值设置为 `false`。

将 `stopAtEntry` 值更改为 `true` 以使调试器在您开始调试时停止在 `main` 方法上。

>**注意**：`preLaunchTask` 设置用于指定要在启动前执行的任务。 确保它与 `tasks.json` 文件 `label` 设置一致。

### 启动调试会话

1. 返回到 `helloworld.cpp`，使其成为活动文件。
2. 按 `kb(workbench.action.debug.start)` 或从主菜单中选择 **Run > Start Debugging**。 在开始逐步浏览源代码之前，让我们花点时间注意用户界面的一些变化：

- 集成终端出现在源代码编辑器的底部。 在 **Debug Output** 选项卡中，您会看到指示调试器已启动并正在运行的输出。
- 编辑器突出显示 `main` 方法中的第一条语句。 这是 C++ 扩展自动为您设置的断点：

   ![Initial breakpoint](images/mingw/stopAtEntry.png)

- 左侧的运行视图显示调试信息。 您将在本教程后面看到一个示例。

- 在代码编辑器的顶部，会出现一个调试控制面板。 您可以通过抓住左侧的点来在屏幕上移动它。

## 单步执行代码

现在您已准备好开始单步执行代码。

1. 单击或按调试控制面板中的 **Step over** 图标。

   ![Step over button](images/cpp/step-over-button.png)

   这会将程序执行推进到 for 循环的第一行，并跳过在创建和初始化 `msg` 变量时调用的 `vector` 和 `string` 类中的所有内部函数调用。 请注意左侧 **Variables** 窗口中的变化。

   ![Debugging windows](images/wsl/debug-view-variables.png)

   在这种情况下，错误是意料之中的，因为尽管循环的变量名现在对调试器可见，但语句尚未执行，因此此时没有可读取的内容。 但是，`msg` 的内容是可见的，因为该语句已经完成。

1. 再次按 **Step over** 前进到该程序中的下一条语句（跳过所有为初始化循环而执行的内部代码）。 现在，**Variables** 窗口显示有关循环变量的信息。
1. 再次按 **Step over** 以执行 `cout` 语句。 （请注意，从 2019 年 3 月版本开始，C++ 扩展在循环退出之前不会将任何输出打印到**调试控制台**。）
1. 如果您愿意，可以一直按 **Step over** 直到向量中的所有单词都打印到控制台。 但是，如果您好奇，请尝试按 **Step Into** 按钮来逐步浏览 C++ 标准库中的源代码！

   ![Breakpoint in gcc standard library header](images/cpp/gcc-system-header-stepping.png)

   要返回您自己的代码，一种方法是按住 **Step over**。 另一种方法是在代码编辑器中切换到 `helloworld.cpp` 选项卡，将插入点放在循环内的 `cout` 语句的某处，然后按 `kb(editor.debug.action .toggleBreakpoint)`。 左侧的装订线中出现一个红点，表示该行已设置断点。

   ![Breakpoint in main](images/cpp/breakpoint-in-main.png)

   然后按 `kb(workbench.action.debug.start)` 从标准库头中的当前行开始执行。 执行将在 `cout` 上中断。 如果你愿意，你可以再次按下 `kb(editor.debug.action.toggleBreakpoint)` 来关闭断点。

   循环完成后，您可以在集成终端中看到输出，以及 GDB 输出的一些其他诊断信息。

   ![Debug output in terminal](images/mingw/mingw-debug-output.png)

## 设置断点

有时您可能希望在程序执行时跟踪变量的值。 您可以通过在变量上设置 **watch** 来做到这一点。

1. 将插入点放在循环内。 在 **Watch** 窗口中，单击加号并在文本框中输入 `word`，这是循环变量的名称。 现在，在您逐步完成循环时查看 Watch 窗口。

   ![Watch window](images/cpp/watch-window.png)

1. 通过在循环之前添加以下语句来添加另一个监视：`int i = 0;`。 然后，在循环内，添加以下语句：`++i;`。 现在像上一步一样为“i”添加一个监视。

1. 要在断点处暂停执行时快速查看任何变量的值，您可以将鼠标指针悬停在该变量上。

   ![Mouse hover](images/cpp/mouse-hover.png)

## C/C++ 配置

如果您想更好地控制 C/C++ 扩展，您可以创建一个 `c_cpp_properties.json` 文件，它允许您更改编译器路径、包含路径、C++ 标准等设置（默认为 C++17 ）， 和更多。

您可以通过从命令面板 (`kb(workbench.action.showCommands)`) 运行命令 **C/C++: Edit Configurations (UI)** 来查看 C/C++ 配置 UI。

![Command Palette](images/cpp/command-palette.png)

这将打开 **C/C++ 配置** 页面。 当您在此处进行更改时，VS Code 会将它们写入 `.vscode` 文件夹中名为 `c_cpp_properties.json` 的文件中。

在这里，我们将 **Configuration name** 更改为 **GCC**，将 **Compiler path** 下拉菜单设置为 g++ 编译器，并设置 **IntelliSense mode** 以匹配编译器（**gcc -x64**)

![Command Palette](images/mingw/intellisense-configurations-mingw.png)

Visual Studio Code 将这些设置放在 `.vscode\c_cpp_properties.json` 中。 如果您直接打开该文件，它应该如下所示：

```json
{
    "configurations": [
        {
            "name": "GCC",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ],
            "windowsSdkVersion": "10.0.18362.0",
            "compilerPath": "C:/msys64/mingw64/bin/g++.exe",
            "cStandard": "c17",
            "cppStandard": "c++17",
            "intelliSenseMode": "windows-gcc-x64"
        }
    ],
    "version": 4
}
```

如果您的程序包含不在工作区或标准库路径中的头文件，您只需添加到 **Include path** 数组设置。

### 编译器路径

该扩展使用 `compilerPath` 设置来推断 C++ 标准库头文件的路径。 当扩展程序知道在哪里可以找到这些文件时，它可以提供智能完成和**转到定义**导航等功能。

C/C++ 扩展会尝试根据在您的系统上找到的默认编译器位置填充 `compilerPath`。 该扩展在几个常见的编译器位置中查找。

`compilerPath` 的搜索顺序是：

* 首先检查 Microsoft Visual C++ 编译器
* 然后在 Windows Subsystem for Linux (WSL) 上寻找 g++
* 然后为 Mingw-w64 使用 g++。

如果您安装了 Visual Studio 或 WSL，您可能需要更改 `compilerPath` 以匹配您项目的首选编译器。 例如，如果您使用 i686 架构、Win32 线程和 sjlj 异常处理安装选项安装 Mingw-w64 版本 8.1.0，则路径将如下所示：`C:\Program Files (x86)\mingw-w64\i686 -8.1.0-win32-sjlj-rt_v6-rev0\mingw64\bin\g++.exe`。

## 故障排除

### 安装了MSYS2，但是还是找不到g++和gdb

您必须按照 [MSYS2 网站](https://www.msys2.org/) 上的步骤并使用 MSYS CLI 安装包含这些工具的 Mingw-w64。

### MinGW 32 位

如果您需要 32 位版本的 MinGW 工具集，请参阅 MSYS2 wiki 上的 [下载](https://www.msys2.org/wiki/MSYS2-installation/) 部分。 它包括指向 32 位和 64 位安装选项的链接。

## 下一步

- 探索 [VS Code 用户指南](/docs/editor/codebasics.md)。
- 查看 [C++ 扩展概述](/docs/languages/cpp.md)。
- 创建一个新的工作区，将你的 `.vscode` JSON 文件复制到其中，为新的工作区路径、程序名称等调整必要的设置，然后开始编码！
