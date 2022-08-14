# bazel 构建 grpc 项目
> 本项目包含了两个 bazel 项目
> 第一个项目是根据 grpc/example 里的 hello world 案例进行了修改，给出了一种用 bazel 构建 grpc、protobuf 的方法
> 第二个是简单的使用 glog 和 gflags 的案例

## 1. 获取 bazelisk
- 具体细节可以参照 bazel 官网关于 bazelisk 的介绍 -- https://bazel.build/install/bazelisk
- mac 推荐直接用 brew install
- 其他版本可以直接去 -- https://github.com/bazelbuild/bazelisk/releases
- v1.12.0 版本内容如下：
```bash
├── bazelisk-darwin
├── bazelisk-darwin-amd64
├── bazelisk-darwin-arm64
├── bazelisk-linux-amd64
├── bazelisk-linux-arm64
├── bazelisk-windows-amd64.exe
├── Source code (zip)
└── Source code (tar.gz)
```
- 下载到 linux 的 `～/` 目录即可，改名为 `bazelisk`，移动到 `/usr/bin` 目录下，可能需要可执行权限
```bash
# 如果 ll 发现下载的文件没有可执行权限 -- 以 bazelisk-linux-amd64 为例：
chmod +x bazelisk-linux-amd64
# 改名，移动到 PATH 目录下
mv bazelisk-linux-amd64 /usr/bin/bazelisk
```
- 执行 `bazelisk version` 命令，它会默认下载 `latest` 版本的 bazel ，有可能会报错(没有科学上网之类的原因)
- 报错的另一个原因是缺少 WORKSPACE 和 .bazelversion 文件

## 2. 工作目录
> 本项目的工作目录如下:
```bash
.
├── README.md
├── grpc_helloword
│   ├── BUILD
│   ├── Makefile
│   ├── WORKSPACE
│   └── src
│       ├── greeter_client.cc
│       ├── greeter_server.cc
│       └── proto
│           └── helloworld.proto
└── logs_flags
    ├── BUILD
    ├── Makefile
    ├── WORKSPACE
    └── main.cc
```
### 2.1 .bazelversion 文件
- 该文件里只需要些一个数字即可，可用版本可以去 bazel 官网查看 -- https://bazel.build/
```bash
cat grpc_helloword/.bazelversion 
5.2.0
```
- 它存在的意义是告诉 bazelisk 用哪个版本来构建项目，直接修改这个数字就行

### 2.2 WORKSPACE 文件
- 该文件主要是命名工作空间、拉取外部依赖文件、拉取外部依赖文件的依赖
```python
workspace(name = "xxxxxx") # 命名工作空间
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive") # bazel 的 http_archive 方法可以拉取远程依赖
# 使用 http_archive 方法
http_archive(
    name = "com_github_grpc_grpc", # 命名拉取的包
    urls = ["https://github.com/grpc/grpc/archive/master.tar.gz"], # 给定路径，一般是 "{github项目网址}/archive/{版本}.tar.gz"
    strip_prefix = "grpc-master", # "{github项目名称}-{版本号}"
)
# 拉取该项目的依赖项
load("@com_github_grpc_grpc//bazel:grpc_deps.bzl", "grpc_deps")
grpc_deps()
# ... 后续一样，把所有的项目拉取一遍
```
### 2.3 BUILD 文件
- BUILD 文件一般是放在一个模块的外层，里面用 bazel 的 rule 语句定义了要编译出来的各种语句
- 比如 cc_binary 表示要将内容编译成可执行文件，具体的细节可以看官方文档 -- https://bazel.build/reference/be/c-cpp
- bazel 编译的语句大概是
```bash
bazelisk build :all # 在 WORKSPACE 文件所在路径下运行该命令，且 BUILD 文件也在该目录下
bazelisk build //src:test # 在 WORKSPACE 文件所在路径下运行该命令，且 BUILD 文件在 src 目录下
# :后面跟着的是 target 名字，all 表示编译所有
# // 后面跟着的是相对 WORKSPACE 的路径，表示 BUILD 文件所在位置
# 具体的使用 bazel 的语法可以参考官网 -- https://bazel.build/concepts/build-ref
```
### 2.5 makefile
- 这里的 makefile 只是把 bazel 命令封装了起来，用户只需要运行：
    - `make update` : 编译整个项目 -- 增量式的
    - `make clean` : 还原编译前的内容
    - `make rebuild` : 重新编译