# Lab0 GitLab 实验报告

25300980042 Jacky

仓库地址：<https://github.com/Jacky108222/ICS-25300980042>

## 一、文档中的问题

### 1. 之前有过多人协同开发的经历吗？

有。之前课程大作业我们小组四个人一起写一个项目，一开始真的是靠微信群传压缩包，文件名后面手动加 v1、v2、final、final2 这种后缀，有一次组员用错了旧版本把别人写好的代码覆盖了，翻了半天聊天记录才找回来。后来有同学提议改用 GitHub，每人开一个分支，写完发 Pull Request，组长看过再合并，之后就再没出过覆盖的事故。算是被土办法坑过一次之后，才真正体会到版本管理的好处。

### 2. Git 为什么要设计"暂存-提交"两个步骤？

说实话做实验之前我觉得这两步挺多余的，直接 save + commit 不行吗。做完这次实验，尤其是解决完冲突之后，我的理解是：

- 工作区里的改动经常很零散。比如我改 main.cpp 的时候可能顺手动过别的文件，有了暂存区，可以只 `git add` 我想放进这次提交的那部分，保证一个 commit 只做一件事，历史才干净。
- commit 记录的是暂存区那一刻的快照，是一个确定的时间点，出了问题可以放心回退。如果工作区随便什么中间状态都能直接进历史，这个历史就不可信了，也没法作为回滚的依据。
- merge 的时候体会更深：解决完冲突还要再 `git add` 一次才能提交，说明连合并的结果也要经过暂存区确认，而不是直接写进去。

打个比方的话，工作区是草稿纸，暂存区是誊好准备装订的那一页，commit 就是装订进册子的一页正式记录。

### 3. `git branch` 和 `git branch -a` 的区别？

`git branch` 只列出本地分支，当前所在的分支前面有个 `*` 号；`git branch -a` 会把远程跟踪分支（remote-tracking branch，形如 `remotes/origin/main`）也一起列出来。远程跟踪分支可以理解成本地对远端分支状态的一份只读"缓存"，在 `git fetch` / `git pull` 的时候更新，它不一定等于远端的最新状态。

做实验的时候我跑了一下，输出是这样：

```text
$ git branch
* feature
  main

$ git branch -a
* feature
  main
  remotes/origin/HEAD -> origin/main
  remotes/origin/main
```

## 二、两篇文章的概括 & 为什么要学 Git

我选读的是 Commit Message 规范和语义化版本这两篇。

### Commit Message 规范（阮一峰）

这篇文章讲的是怎么把提交说明写得规范。提交信息分成标题和正文两部分：标题一行写完、用祈使语气、结尾不加句号；正文写清楚为什么这么改。文章重点介绍了 Angular 团队的约定：commit message 写成 `type(scope): subject` 的形式，type 有 feat（新功能）、fix（修 bug）、docs（文档）、style、refactor、test、chore 等等。好处是历史可读、能配合工具自动生成 change log，而且看 type 就知道每次提交大概干了什么。对照我自己之前写的 "update" "修改" 这种提交信息，确实差距很大。

### 语义化版本（Semantic Versioning）

这篇讲的是版本号的规则：`MAJOR.MINOR.PATCH`。做了不兼容的 API 改动就升主版本号，向下兼容地加新功能升次版本号，只修 bug 升修订号，另外还有 `1.0.0-alpha` 这类预发布版本和构建元信息的写法。核心思想是让版本号本身带上"机器可读"的兼容性信息，包管理器据此判断能不能自动升级，开发者看到主版本号跳了就知道要去查迁移指南。以前只觉得版本号是个名字，看完才发现它其实是一种约定出来的通信协议。

### 为什么要学 Git

我的体会是：Git 不是一个"存代码的备份盘"，而是一套项目里的沟通语言。commit message 是对协作者（也包括几个月后的自己）解释"为什么这么改"；语义化版本是对下游用户承诺"这次改动影响多大"；分支模型约定的是团队怎么并行干活。这门课后面的实验都是一环扣一环往下迭代的，学会了 Git 才能放心实验、大胆回退，而不是靠复制整个文件夹来"保平安"。学 Git 其实是在学工程化协作的思维方式。

## 三、实验步骤

环境：Windows 11 + PowerShell，g++ (MSYS2) 16.1.0。

关于语言有一点说明：我平时写 C++ 比较多，所以把 `main.c` 重写成了 `main.cpp`（用 `iostream` 输出），Makefile 也相应把 `gcc` 换成了 `g++`。另外 Windows 下没有 make，我直接用 `g++ -Wall -O2 -o main main.cpp` 编译，和 Makefile 里的命令是一样的。

### 1. 建仓库

用文档说的方法，从模板仓库 <https://github.com/ICS-26Fall-FDU/GitLab> 点 `Use this template` → `Create a new repository` 建了自己的仓库，然后 clone 到本地：

```text
$ git clone https://github.com/Jacky108222/ICS-25300980042.git
```

### 2. 换成 C++ 并完成 TODO

先重写成 C++ 版本（`main.c` → `main.cpp`，Makefile 用 `g++`）：

```makefile
CXX = g++
CXXFLAGS = -Wall -O2
TARGET = main
SRCS = main.cpp
OBJS = $(SRCS:.cpp=.o)

$(TARGET): $(OBJS)
	$(CXX) $(CXXFLAGS) -o $(TARGET) $(OBJS)

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

.PHONY: clean
clean:
	rm -f $(OBJS) $(TARGET)
```

然后把 TODO 的输出改成自己的信息：

```cpp
#include <iostream>

int main()
{
    // @TODO: print a sentence you want.
    std::cout << "Hello, ICS 26Fall! I am Jacky, student ID 25300980042." << std::endl;
    return 0;
}
```

提交并编译验证：

```text
$ git add main.cpp
$ git commit -m "complete the TODO in main.cpp"
[main 73dc31c] complete the TODO in main.cpp
 1 file changed, 1 insertion(+), 1 deletion(-)

$ g++ -Wall -O2 -o main.exe main.cpp
$ ./main.exe
Hello, ICS 26Fall! I am Jacky, student ID 25300980042.
```

### 3. 分支与合并冲突

新建 feature 分支，把 main.cpp 里的输出改成 "Hello from the feature branch." 并提交：

```text
$ git checkout -b feature
Switched to a new branch 'feature'

$ git add main.cpp
$ git commit -m "modify main.cpp in the feature branch"
[feature 4c315a3] modify main.cpp in the feature branch
 1 file changed, 1 insertion(+), 1 deletion(-)
```

切回 main，把**同一行**改成 "Hello from the main branch." 再提交（改同一行是关键，不然 Git 会自动合并不报冲突）：

```text
$ git checkout main
$ git add main.cpp
$ git commit -m "modify main.cpp in the main branch too"
[main 05b56c8] modify main.cpp in the main branch too
 1 file changed, 1 insertion(+), 1 deletion(-)
```

然后合并，冲突如预期出现了：

```text
$ git merge feature
Auto-merging main.cpp
CONFLICT (content): Merge conflict in main.cpp
Automatic merge failed; fix conflicts and then commit the result.
```

打开 main.cpp，Git 用标记把两边的改动都摆出来了：

```cpp
<<<<<<< HEAD
    std::cout << "Hello from the main branch." << std::endl;
=======
    std::cout << "Hello from the feature branch." << std::endl;
>>>>>>> feature
```

`<<<<<<< HEAD` 到 `=======` 是 main 这边的，`=======` 到 `>>>>>>> feature` 是 feature 那边的。我把两边的意思合成一句，删掉标记：

```cpp
#include <iostream>

int main()
{
    // @TODO: print a sentence you want.
    std::cout << "Hello from both branches - I resolved the conflict myself." << std::endl;
    return 0;
}
```

再 add + commit 完成合并，编译确认没问题：

```text
$ git add main.cpp
$ git commit -m "merge feature into main and resolve the conflict"
[main 0377f7c] merge feature into main and resolve the conflict

$ g++ -Wall -O2 -o main.exe main.cpp
$ ./main.exe
Hello from both branches - I resolved the conflict myself.
```

最后的提交历史，能清楚看到 feature 分支分出去又合回来：

```text
$ git log --oneline --graph
*   0377f7c merge feature into main and resolve the conflict
|\
| * 4c315a3 modify main.cpp in the feature branch
* | 05b56c8 modify main.cpp in the main branch too
|/
* 73dc31c complete the TODO in main.cpp
* c5320cf rewrite the program in C++ (main.cpp + g++ Makefile)
* 8777805 Initial commit
```

## 四、踩过的坑

- 第一次 merge 之前我以为随便改两下就能出冲突，后来发现如果两次改动不在同一行附近，Git 会直接自动合并。要让两边改**同一处**才会真冲突。
- push 的时候一度连不上 GitHub，报 `Failed to connect to github.com port 443 via 127.0.0.1`，查了半天发现是我 git 全局配了本地代理（`git config --global http.proxy`）但代理软件没开，开了就好了。
- Windows 上没有 make，一开始还挺纠结要不要装，后来发现直接用 g++ 命令行编译也是一样的。

## 五、建议（可选）

- 文档 merge 那一节可以配一张 `git log --graph` 的示意图，新手看分支分叉汇合的图比看文字描述直观很多。
- 可以在文档里明确一下 Windows 用户怎么处理没有 make 的问题（直接用 gcc/g++ 也行）。
