[![CI badge](https://github.com/micropython/micropython/workflows/unix%20port/badge.svg)](https://github.com/micropython/micropython/actions?query=branch%3Amaster+event%3Apush) [![codecov](https://codecov.io/gh/micropython/micropython/branch/master/graph/badge.svg?token=I92PfD05sD)](https://codecov.io/gh/micropython/micropython)

Micropython项目
=======================
<p align="center">
  <img src="https://raw.githubusercontent.com/micropython/micropython/master/logo/upython-with-micro.jpg" alt="MicroPython Logo"/>
</p>

Micropython是可以运行在单片机和小型嵌入式系统上的python3
Micropython项目官网是 [micropython.org](http://www.micropython.org).

警告:该项目处于测试阶段，所有东西都有可能更改。

Micropython满足Python3.4的所有特性 (包括 异常,
`with`, `yield from`等等, 还有来自Python3.5的关键字 `async`/`await`). 并且提供以下数据类型: `str` (包括基础Unicode支持), `bytes`, `bytearray`, `tuple`, `list`, `dict`, `set`,
`frozenset`, `array.array`, `collections.namedtuple`, `class`, 实例.
内置模块包括 `sys`, `time`, 以及 `struct`等等. Select ports have
support for `_thread` module (multithreading). Note that only a subset of
Python 3 functionality is implemented for the data types and modules.

Micropython可以以文本或预编译(mpy)形式执行python代码，无论是文本形态的python代码还是预编译的字节码，都可以在"文件系统"里执行或者在编译micropython固件前嵌入到micropython源码里执行。

你可以看看 http://github.com/micropython/pyboard 这个仓库，里面有pyboard开发板的例程和参考资料，pyboard是Micropython官方支持的开发板


此仓库的主要组件
- py/ -- the core Python implementation, including compiler, runtime, and
  core library.
- mpy-cross/ -- the MicroPython cross-compiler which is used to turn scripts
  into precompiled bytecode.
- ports/unix/ -- a version of MicroPython that runs on Unix.
- ports/stm32/ -- a version of MicroPython that runs on the PyBoard and similar
  STM32 boards (using ST's Cube HAL drivers).
- ports/minimal/ -- a minimal MicroPython port. Start with this if you want
  to port MicroPython to another microcontroller.
- tests/ -- test framework and test scripts.
- docs/ -- user documentation in Sphinx reStructuredText format. Rendered
  HTML documentation is available at http://docs.micropython.org.

附加组件:
- ports/bare-arm/ -- ARM MCU上的最小版本，主要用在需要控制大小的场合
- ports/teensy/ -- 运行于Teensy3.1的Micropython
  (能用但不算完善)
- ports/pic16bit/ -- 用于16位 PIC的micropython版本
- ports/cc3200/ -- 在TI CC3200上运行的micropython版本
- ports/esp8266/ -- 在ESP8266 SoC上运行的micropython版本
- ports/esp32/ -- 在ESP32上运行的micropython版本
- ports/nrf/ -- 在Nordic's nRF51 和 nRF52上运行的micropython版本
- extmod/ -- C语言实现的附加python模块
- tools/ -- 各种工具，包括pyboard.py模块
- examples/ -- 一些python例程

上面的目录里可能包含README
用"MAKE"命令构建这些组件, 或者BSD系系统使用的"gmake"
你还需要bash，gcc，python3.3+
(如果你的系统只有python2.7，在调用make指令的时候应该加入选项`PYTHON=python2`).

Micropython的交叉编译器, mpy-cross
-----------------------------------------

在你构建大多数ports里的各种micropython版本之前，你需要先构建mpy-cross，这玩意用来将.py文件编译为适应的.mpy文件，如果你没有先编译mpy-cross，编译MP的时候不会顺利。

    $ cd mpy-cross
    $ make

Unix版本
----------------
UNIX port需要一个带有GNU make和GCC的标准UNIX环境，按照人话来说就是:只要你的linux有make和gcc，剩下的就好说了
它支持x86和x64架构，也就是俗称的32位和64位，更严谨一点是X86-64和X86-32，同时也支持ARM和MIPS架构。如果你要把micropython移植到另一个又操蛋又冷门的架构，你可能需要为异常处理和垃圾回收(gc)额外写一些汇编代码

构建！ (如果你想知道依赖项，下面的"外部依赖"章节写了):

    $ cd ports/unix
    $ make submodules
    $ make

编译完成，试一试:

    $ ./micropython
    >>> list(5 * x + y for x in range(10) for y in [4, 2, 1])

用`CTRL-D` (即EOF) 来退出这个shell
了解命令行参数

    $ ./micropython -h

运行一个完整的测试程序

    $ make test

UNIX版本的micropython解释器里自带一个upip包管理器，你可以像在电脑上使用python-pip一样使用它(你本来就是)，下面是例子:

    $ ./micropython -m upip install micropython-pystone
    $ ./micropython -m pystone

你可以在这里看适用于micropython的可用模块并安装它
[PyPI](https://pypi.python.org/pypi?%3Aaction=search&term=micropython).
标准模块来自于这里:
[micropython-lib](https://github.com/micropython/micropython-lib)

外部依赖
---------------------

构建micropython可能需要一些依赖项
UNIX版本依赖于`libffi`运行库和 `pkg-config`工具 在debian系操作系统上, 你只需要用apt安装 `build-essential`
(这个包包括一堆工具链和make本身，可以说是万金油了), `libffi-dev`, 和 `pkg-config`三个包即可

如果你要编译其它版本的micropython，你可以在那个版本的"ports"目录里的READE.md里找到依赖项的细节(例如 `ports/unix/`) 然后先执行:

    $ make submodules
    
你可以用这个指令时不时更新一下仓库 然后执行:

    $ make deplibs

This will build all available dependencies (regardless whether they
are used or not). If you intend to build MicroPython with additional
options (like cross-compiling), the same set of options should be passed
to `make deplibs`. To actually enable/disable use of dependencies, edit
`ports/unix/mpconfigport.mk` file, which has inline descriptions of the options.
For example, to build SSL module (required for `upip` tool described above,
and so enabled by default), `MICROPY_PY_USSL` should be set to 1.

For some ports, building required dependences is transparent, and happens
automatically.  But they still need to be fetched with the `make submodules`
command.

STM32版本
-----------------
STM32版本需要一个ARM编译器, arm-none-eabi-gcc, 和相关的bin-utils.  如果你使用Arch Linux, 你只需要安装 arm-none-eabi-binutils,
arm-none-eabi-gcc arm-none-eabi-newlib 这三个包.  不是的话，请在这尝试:
https://launchpad.net/gcc-arm-embedded

编译:

    $ cd ports/stm32
    $ make submodules
    $ make

然后你需要让你的板子进入DFU模式. 如果你使用pyboard(恰饭), 短接3V3到P1/DFU接口即可进入DFU模式。

然后通过USB DFU将固件写入其中

    $ make deploy

这条指令会使用内置的 `tools/pydfu.py` 脚本.  如果无法正常烧写固件，可能是因为你没有权限, 请尝试使用 `sudo make deploy`.
See the README.md file in the ports/stm32/ directory for further details.

贡献
------------

MicroPython is an open-source project and welcomes contributions. To be
productive, please be sure to follow the
[Contributors' Guidelines](https://github.com/micropython/micropython/wiki/ContributorGuidelines)
and the [Code Conventions](https://github.com/micropython/micropython/blob/master/CODECONVENTIONS.md).
Note that MicroPython is licenced under the MIT license, and all contributions
should follow this license.
