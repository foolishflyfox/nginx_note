# Nginx 的构建

Nginx 构建文件通过 http://nginx.org/en/download.html 下载。

构建命令为：
```shell
$ ./configure \
--sbin-path=/usr/local/nginx/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--pid-path=/usr/local/nginx/nginx.pid \
--with-http_ssl_module \
--with-pcre=../pcre-8.38 \
--with-zlib=../zlib-1.2.8
```
配置完成以后，使用 make 命令编译 nginx。

可用参数：
- `sbin-path`: 生成的二进制文件存放文件夹。
- `builddir`: 指定生成文件的中间文件的路径(用`NGX_OBJS`存储)，默认为 *objs* 文件夹。
- `with-cc`: 指定编译器名称，例如在 Windows 下需要显示指定 `--with-cc=cl`，其值将用于配置变量 `$CC`。
- `crossbuild`: 交叉编译，指定宿主平台。
- `with-cc-opt`: 指定附加的参数，例如 `--with-cc-opt=-DFD_SETSIZE=1024` 表示在编译时添加选项 `-DFD_SETSIZE=1024`。

## configure 文件分析

1. `. auto/options` 根据用户的传入参数设置环境变量；
1. `. auto/init` 初始化项目，如初始化 Makefile 文件，设置自动生成的头文件路径，错误信息记录文件。
1. `. auto/sources` 项目配置变量记录源码的文件位置。
1. `. auto/cc/conf`: 测试各种语法特性能否使用，如测试 C99宏命令、测试原子操作等。
    - `. auto/cc/name`: 测试编译器是否可用，确定编译器名
    - `. auto/cc/gcc`(Linux) / `. auto/cc/clang`(Darwin)

## auto 文件夹

配置文件。

### cc

#### name

c 编译器的类型：
- *cc*: 不同平台上实际的名字不同
    - `icc`: Intel C++ compiler, `cc -v` 中含有 `Intel(R) C`;
    - `gcc`: GNU C compiler, `cc -v` 中含有 `gcc version`;
    - `clang`: Clang C compiler(Mac OS X下), `cc -v` 含有 `clang version` 或 `LLVM version`;
    - `sunc`: Sun C compiler, `cc -v` 含有 `Sun C`;
    - `ccc`: Compaq C compiler(康柏，已被惠普收购), `cc -v` 中一行以 `Compaq C` 开头;
    - `acc`: HP aC++ compiler, `cc -v` 中一行以 `aCC` 开头;
- *bcc32*: Borland C++ compiler
- *cl*: Microsoft Visual C++ compiler
- *wcl386*: Open watcom C compiler

*name* 用于测试 C 编译器的类型，并设置 `NGX_CC_NAME` 为编译系统中的编译器的名字。

### feature

测试变量 `$ngx_feature`。需要先指定如下的变量：
- `ngx_feature`: 
- `ngx_feature_name`: 在 *feature* 会将该变量的小写变为大小；
- `ngx_feature_run`: 是否进行运行测试，`yes` 表示需要编译、运行测试；`value` 表示需要编译、运行，并将生成的可执行文件输出信息写入到 *$NGX_OBJS/ngx_auto_config.h* 文件中；`bug` 表示测试是否有bug，行为刚好与 `yes` 相反。
- `ngx_feature_incs`: 
- `ngx_feature_path`: 路径用空格分开，变为 `-I path1 -I path2` 的形式；
- `ngx_feature_libs`: 
- `ngx_feature_test`: 指定测试语句

### options 文件


### sources 文件

## 自动生成的文件

### ngx_auto_config.h

定义配置文件，供编译时使用。

- `NGX_COMPILER`: 编译器名与版本号；


## Nginx 的模块

### 核心模块

核心模块包含3个子模块：`ngx_core_module`、`ngx_errlog_module` 和 `ngx_conf_module`，源码位于 *src/core* 文件夹中。

### 事件模块

时间模块包含两个子模块：`ngx_events_module` 和 `ngx_event_core_module`，源码分别位于 *src/event* 和 *src/event/modules* 文件夹中。


## 参考

- [Nginx 编译详解](https://www.jianshu.com/p/df050fd7d64e)
