# 第 02 课 Linux/WSL2 环境准备与依赖安装（详细教程版）

## 0. 本课定位与结果
这节课的目标不是“看懂命令”，而是“把可复现环境真正搭起来”。

你学完后必须达到：
- 能在 Linux 或 WSL2 中稳定编译 TinyWebServer。
- 能解释每个关键依赖为什么需要安装。
- 遇到常见报错能快速定位并修复。

建议时长：120 到 150 分钟。

## 1. 课前说明

### 1.1 你需要准备什么
- Windows 10/11（推荐 WSL2 Ubuntu 22.04 或 20.04）或原生 Linux。
- 管理员权限（安装软件包需要）。
- 项目源码目录已就绪。

### 1.2 本课学习方法
按“先确认环境、再安装依赖、再做验证、最后记录”的流程执行：
1. 环境基线检查。
2. 编译工具链安装。
3. MySQL 客户端开发库安装。
4. 编译验证。
5. 常见问题排查。

## 2. 环境路径与目标确认

### 2.1 你现在要确认
- 项目目录中有 build.sh、makefile、main.cpp。
- 当前目标不是功能跑通，而是依赖完整、编译无缺库。

### 2.2 你应该知道的关键依赖
本项目链接参数包含 pthread 和 mysqlclient，因此至少需要：
- g++ / make
- pthread（通常随 glibc 提供）
- libmysqlclient 开发库

## 3. 第一步：环境基线检查（15 分钟）

在终端执行以下检查命令（按顺序）：

~~~bash
uname -a
lsb_release -a
which g++
which make
g++ --version
make --version
~~~

你要记录的信息：
- Linux 发行版版本。
- g++ 和 make 是否存在。
- 版本号是否正常输出。

如果 which g++ 或 which make 没输出路径，说明工具链未安装，进入第 4 步。

## 4. 第二步：安装编译工具链（20 分钟）

### 4.1 Ubuntu 推荐命令
~~~bash
sudo apt update
sudo apt install -y build-essential cmake pkg-config
~~~

解释：
- build-essential 提供 g++、make 等核心编译工具。
- cmake 不是本项目硬依赖，但后续扩展常用。
- pkg-config 方便后续检查库信息。

### 4.2 安装后立即验证
~~~bash
which g++
which make
g++ --version
~~~

验收标准：三个命令都返回正常结果。

## 5. 第三步：安装 MySQL 开发依赖（25 分钟）

### 5.1 Ubuntu 22.04 常用安装方式
~~~bash
sudo apt install -y libmysqlclient-dev
~~~

如果提示包不可用，可使用兼容包：
~~~bash
sudo apt install -y default-libmysqlclient-dev
~~~

### 5.2 验证 mysqlclient 已安装
~~~bash
ldconfig -p | grep -i mysqlclient
~~~

有输出即表示系统可见该动态库。

### 5.3 可选：安装 MySQL 服务端（用于后续登录注册实验）
~~~bash
sudo apt install -y mysql-server
sudo systemctl status mysql
~~~

本课不强制配置数据库表，但你需要知道后续课程会用到。

## 6. 第四步：定位项目编译规则（15 分钟）

请打开并观察：
- makefile
- build.sh

你要找出的结论：
1. 可执行文件目标名是什么。
2. 源文件列表来自哪些模块。
3. 链接参数里是否有 -lpthread 和 -lmysqlclient。

标准答案（你自己先找，再核对）：
- 目标文件名是 server。
- 关键源文件覆盖 timer/http/log/CGImysql/webserver/config/main。
- 需要 pthread 和 mysqlclient。

## 7. 第五步：执行首次编译（20 分钟）

进入项目根目录后执行：
~~~bash
sh ./build.sh
~~~

或直接：
~~~bash
make server
~~~

如果编译成功，应看到 server 可执行文件生成。

验证命令：
~~~bash
ls -l server
file server
~~~

## 8. 第六步：最小运行验证（10 分钟）

本课只做最小运行验证，不要求网页登录成功。

~~~bash
./server
~~~

观察点：
- 程序是否启动而不立即崩溃。
- 是否有端口监听（下一课会详细展开）。

若当前机器没有配置数据库，本项目可能在数据库相关初始化阶段报错，这是可预期现象。你需要把报错记录下来，留到第 4 课统一解决。

## 9. 常见错误与排查手册（重点）

### 9.1 错误：mysql/mysql.h: No such file or directory
原因：MySQL 开发头文件未安装。
处理：安装 libmysqlclient-dev 或 default-libmysqlclient-dev。

### 9.2 错误：cannot find -lmysqlclient
原因：链接阶段找不到 mysqlclient 库。
处理：
1. 先执行 ldconfig -p | grep mysqlclient。
2. 若无输出，重新安装开发包。
3. 若仍失败，确认架构与系统库路径一致。

### 9.3 错误：Permission denied: ./build.sh
原因：脚本无执行权限。
处理：
~~~bash
chmod +x build.sh
sh ./build.sh
~~~

### 9.4 错误：make: command not found
原因：未安装 make。
处理：安装 build-essential。

### 9.5 错误：运行后秒退
原因：可能数据库初始化失败、参数异常、环境不一致。
处理：
1. 先确认编译成功。
2. 再检查数据库配置和运行参数。
3. 记录日志与报错原文，不要只说“跑不起来”。

## 10. 学习加深：为什么这些依赖是必须的

请你尝试自己回答：
1. 为什么要链接 pthread。
2. 为什么要链接 mysqlclient。
3. 为什么先做环境课而不是直接讲网络模型。

参考要点：
- 线程池依赖线程能力。
- 登录注册业务依赖数据库访问。
- 环境不稳会让后续所有学习反复卡住。

## 11. 本课实践任务（必须完成）

### 任务 A：环境检查记录
输出一个文档，包含：
- 系统版本
- g++ 版本
- make 版本
- mysqlclient 验证结果

### 任务 B：编译验证记录
输出一个文档，包含：
- 执行了哪些命令
- 成功或失败
- 若失败，错误全文与处理过程

### 任务 C：问题清单
列出你当前机器上仍未解决的问题，至少 2 条。

## 12. 验收标准（严格版）
完成以下全部条件才算通过：
1. g++ 与 make 命令可用。
2. mysqlclient 动态库可检索。
3. 项目可以完成编译。
4. 你有完整的环境与编译记录。
5. 你能解释每个关键依赖的用途。

## 13. 本课作业提交模板

~~~text
作业名称：lesson02-环境准备与依赖安装
系统信息：
执行命令：
关键输出：
遇到问题：
解决方案：
遗留问题：
~~~

## 14. 进阶建议（可选）
- 在 WSL2 之外再准备一套纯 Linux 环境，对比差异。
- 尝试将依赖安装步骤整理为一键脚本。
- 写一个最小自检脚本，自动检查 g++/make/mysqlclient。

## 15. 进入下一课前自测
以下三项都为“是”再进入第 03 课：
- 我可以在 10 分钟内把环境从零检查一遍。
- 我知道编译失败时先看哪里。
- 我能解释为什么本项目必须有 mysqlclient 和 pthread。
