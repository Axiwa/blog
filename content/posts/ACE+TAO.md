---
title: ACE+TAO+OpenDDS安装
categories: Debug
date: 2022-06-30
---

我需要用OpenDDS进行实时的数据处理，使用前要benchmark一下，因此想用自带的performance-tests

系统 Ubuntu 20.04

OpenDDS-3.20

ACE+TAO-7.0.7

**官网：**

最简安装，不需要自己下载编译ACE+TAO，不指定任何configure option。简而言之：下载，解压，进入解压的OpenDDS目录，`./configure`，`make`，`source setenv.sh` 设置环境，测试，成功！

[https://opendds.org/quickstart/GettingStartedLinux.html](https://opendds.org/quickstart/GettingStartedLinux.html)

官方Github的文档，告诉你需要什么feature都告诉configure。比如java支持啊，需要自带tests啊，实际的参数都请去相应的文档查阅

[https://github.com/objectcomputing/OpenDDS/blob/master/INSTALL.md](https://github.com/objectcomputing/OpenDDS/blob/master/INSTALL.md)

---

Make life easier:

configure支持的输入用
```bash
./configure --help
```
CXX标准似乎默认是std=c++11，不放心也可以加上`--std=c++11`

`--tests` 如果是官网下载解压的文件夹，`./configure --tests`会报错，请你用 `--gtest=???` 指定googletest的根目录。比如googletest解压之后有了这个`/usr/local/include/gtest`，那么`--gtest=/usr/local`。如果是Github下载的文件夹，终端 `git submodule init` + `git submodule update`，再用`--tests`即可。具体文档：`$DDS_ROOT/tests/gtest_setup.txt`


**网友：**

ACE+TAO自行安装编译

[Linux系统下OpenDDS安装及测试2021-07-10](https://blog.csdn.net/mrliushuaishuai/article/details/118633577)

感谢网友的保姆级教程！唯一的问题是我不知道在哪指定`no_cxx11=0`导致无法手动在tests中build，以及不小心指定了`make -B`以后所有可以make的全部make，不管怎么`make clean`也调不回来，一整天也没完成 T_T...

---
Make life easier:

如果不按照教程里新建`$ACE_ROOT/ace/config.h`和`$ACE_ROOT/include/makeinclude/platform_macros.GNU`，仅仅下载，解压，改权限，添加环境变量，然后直接在`$DDS_ROOT`中`./configure`的话，将会由它自动添加和修改，可以解决没有用C++11的问题。来自 [My system refuses to configure with c++11 #1842 ](https://github.com/objectcomputing/OpenDDS/issues/1842)

不想无休止地给ACE和TAO的测试目录make怎么办？参考`$TAO_ROOT/TAO-INSTALL.html`中“On UNIX platforms”第4条，重新生成吧

自己怎么指定CXX？ To be continued...

**运行测试**
```bash
cd $DDS_ROOT/tests
./auto_run_tests.pl 
```
test PASSED真是无论何时都那么令人心旷神怡啊

**性能测试**

[https://opendds.readthedocs.io/en/latest/internal/bench2.html](https://opendds.readthedocs.io/en/latest/internal/bench2.html)
