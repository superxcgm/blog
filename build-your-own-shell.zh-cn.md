# 构建你自己的shell

> What I cannot create, I do not understand. - Richard Feynman

重复发明轮子是程序员的传统美德，如果你也想浪费时间，那就看下去吧。

## 什么是Shell

Shell是大多程序员都会用到的一种软件，通过使用键盘敲入指令和系统交互，而不是通过可见即所得（What you see is what you get）的图形化窗口（GUI）。常见的shell一般有[bash](https://zh.wikipedia.org/wiki/Bash), [zsh](https://zh.wikipedia.org/wiki/Z_shell)等。

![zsh](./img/zsh.png)

## 技术栈

语言&标准：C++17

构建工具：CMake

测试：Google Test

代码风格：Google C++

依赖管理：Conan

持续集成：Github Action

## 相关资源

代码：[superxcgm/xcShell](https://github.com/superxcgm/xcShell)

看板：[xcShell](https://github.com/users/superxcgm/projects/4)

在使用代码之前，你可能需要先阅读代码仓库里的README，了解如何构建和运行。

## 结构

Shell大体可以分为以下三个部分：

1. 启动：加载配置文件（如：zsh会在启动时加载`.zshrc`）。
2. REPL：读取用户输入的指令，执行，读取用户输入的指令，执行不断循环。
3. 退出：清理所用到资源，恢复终端设置。

```c++
class XcShell {
 public:
  void init();
  void process();
  int exit();
};
```

我们创建了一个XcShell的类，包含了三个方法，每个方法对应上面提到的一个部分。参数和返回值的部分先简单写了一下，后面还会再改。我们会先关注中间的process部分，其他两部分在后面再考虑。

[相关代码提交](https://github.com/superxcgm/xcShell/pull/7)

## Process

### 读取用户输入

这里我们首先输出`> `提示用户输入，使用`getline`来获取用户输入的行，然后打印用户的输入，再提示用户输入，读用户输入，打印，直到用户输入结束（EOF）。（你可以通过按`Ctrl + D`向程序发送EOF）

```c++
void XcShell::process(std::istream &is, std::ostream &os) {
  while (!is.eof()) {
    os << "> ";
    std::string line;
    getline(is, line);
    os << line << std::endl;
  }
}
```

这里修改了`process`方法的入参，注入了输入/输出，而不是直接依赖`stdin`、 `stdout`，提升可测试性。

Known issue: 这里的line理论上可以支持很大的行，只要你内存管够，但我实际在测试的时候，发现在iTerm2里只能输入1024个字符，但是使用输入重定向到文件的话，就没有这个限制，所以应该不是`string`和`getline`的锅。[issue/14](https://github.com/superxcgm/xcShell/issues/14)

[相关代码提交](https://github.com/superxcgm/xcShell/pull/13)

### 解析用户输入

上一步我们读到的是一整个字符串，所以我们需要将它拆分成两个部分（先不考虑其他功能，如输入输出重定向、管道等）：

1. 命令
2. 参数列表

如`echo hello world`

需要拆分成命令`echo`和参数列表`['hello', 'world']`。

```c++
std::tuple<std::string , std::vector<std::string>>
XcShell::parseUserInput(const std::string &str) {
  auto parts = xc_utils::split(str);
  std::string command = parts[0];
  parts.erase(parts.begin());
  return {command, parts};
}
```



```c++
std::vector<std::string> xc_utils::split(const std::string& str) {
  std::stringstream ss(str);
  std::vector<std::string> parts;

  while (ss) {
    std::string part;
    ss >> part;
    parts.push_back(part);
  }

  if (parts[parts.size() - 1] == "") {
    parts.pop_back();
  }

  return parts;
}
```

这里简单地使用了`stringstream`去拆分输入，使用空格作为分割符（这里没有处理引号，`"hello world"`会被拆分成`"hello`和`world"`，会在后面去支持）。

[相关代码提交](https://github.com/superxcgm/xcShell/pull/15)

### 执行

placeholder

### 内建命令支持：`cd`

placeholder

### 显示当前所在目录

placeholder
