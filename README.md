# C Toolbox

Routilet 维护的独立 C99 工具箱，面向已经熟悉 C 的使用者，提供字符串、容器、IO 和按需源码装配。

所属组织：[Routilet](https://github.com/Routilet) · 原始设计者：Annouimo。

v0.11.0 默认生成单头文件，提交时按需组装。相比面向课程基础编程的 [C Exam Kit](https://github.com/Routilet/c-exam-kit)，C Toolbox 提供更丰富的容器和模块化 API，独立使用，不依赖 Companion。

## 使用

从 [GitHub Releases](https://github.com/Routilet/c-toolbox/releases/latest) 下载单文件 Python 入口。运行后输入题目目录，再选择程序框架。空输入直接导出 Markdown 手册。默认生成 `main.c`、`c_toolbox.h` 和 `prepare.py`。选择“仅创建工具箱”可省略 `main.c`。

默认模式面向单个 `main.c`，支持内置 API，以及工具箱和标准库的 `#include`。运行 `prepare.py` 时，它读取 `main.c`，用内置索引组装单文件，报告字节数，编译通过后复制。完整头文件用于本地编译，须和 `prepare.py` 成对更新。

需要自定义容器、用户头文件或其他预处理指令时，用入口脚本的 `--modular` 生成分模块项目。该模式在提交时展开本地头文件。两种模式均可用 `--no-copy` 只检查和保存，或 `--run` 显式运行。每题保留创建时的版本。

## 开发

```sh
python toolbox/build_headers.py
python toolbox/build.py
python toolbox/tests/test_modular.py
python toolbox/tests/test_toolbox.py
python toolbox/tests/test_workflow.py
```

新公共头文件在 toolbox/include；实现模板在 toolbox/templates。迁移状态与约定见 toolbox/docs/modular-migration.md。旧生成器暂作回归对照，旧脚手架当前目录已移除，完整历史保留。

## 最短用法

```c
#include "c_toolbox.h"
int main(void) {
    int a = read_int();
    int b = read_int();
    print_ll((long long)a + b, "\n");
    return 0;
}
```

所有权、模块表和接口以[速查手册](toolbox/docs/quick_reference.md)为准。

## 发布

合并到 main 后，自动运行 Linux GCC/Clang、macOS、Windows MinGW 和 Python 3.9 检查，构建唯一的版本化 Python 附件。新版本先创建草稿，上传并验证附件后自动发布正式 Release；失败保留草稿供检查。已发布版本自动跳过，不覆盖附件；修复时增加版本号。不提供 PDF 或可执行二进制。

## 项目来源与授权

本项目源于原 Annouimo/c-exam-kit 仓库的工具箱开发分支，保留原提交历史；现由 Routilet 独立维护。授权范围见 [LICENSING.md](LICENSING.md)。
