# CPython源码阅读之五: 配置与输入



There are many ways Python code can be run in CPython. Here are some of the most commonly used:
- 1. By running python -c and a Python string
- 2. By running python -m and the name of a module
- 3. By running python [file] with the path to a ﬁle that containsPython code
- 4. By piping Python code into the python executable over stdin, e.g.,cat [file] | python
- 5. By starting a REPL and executing commands one at a time
- 6. By using the C API and using Python as an embedded environment



在 Python 中有三个不同的配置阶段,分别用于不同的目的。
以下是这三种配置的详细解释及其配置的项目：

1. 预初始化配置（Pre-Initialization Configuration）
预初始化配置主要用于在 Python 解释器完全启动之前设置必要的配置选项。这些配置通常在解释器初始化的早期阶段确定,确保在后续阶段有适当的设置。常见的项目包括：
- UTF-8模式 (utf8_mode)：设置是否启用 UTF-8 模式,影响字符编码的处理。
- 开发模式 (dev_mode)：设置是否启用开发模式,影响错误处理和调试信息的输出。
- 隔离模式 (isolated)：控制脚本的运行环境,决定是否使用用户的 site-packages。
- 使用环境变量 (use_environment)：决定是否从环境变量读取配置。
- 本地化设置 (configure_locale)：用于设置字符类别和字符处理方式的本地化。

1. 运行时配置（Runtime Configuration）
运行时配置是在 Python 解释器启动后,根据用户的输入、环境变量等动态配置的选项。运行时配置主要包括：
- 命令行参数解析 (parse_argv)：是否解析命令行参数,以控制解释器的行为。
-缓冲标准输入输出 (buffered_stdio)：控制标准输入输出是否缓冲。
- 详细模式 (verbose)：控制输出的详细程度,通常用于调试和信息记录。
- 跳过源代码第一行 (skip_source_first_line)：决定是否跳过源代码的第一行,常用于某些特定的运行要求。
- 
1. 构建配置（Build Configuration）
构建配置是涉及到 Python 解释器编译和链接时的设置,通常在构建阶段就已经确定。这些配置决定了 Python 解释器的各种底层功能和标准库的支持。常见的项目包括：
- 内存分配器 (allocator)：指定使用哪种内存管理策略,通过环境变量进行配置。
- 系统特定编码 (legacy_windows_fs_encoding)：在 Windows 系统上,决定是否使用传统的文件系统编码。
- 扩展模块的支持：决定是否编译某些 C 扩展模块。




---

> 作者: 迷途小狼崽  
> URL: http://localhost:1313/posts/python/cpython_internals_5/  

