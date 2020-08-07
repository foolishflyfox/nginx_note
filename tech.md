# 技术总结

## 脚本技术

### 设置环境变量

```shell
LC_ALL=C
export LC_ALL
```
其中 `LC_ALL=C` 在该脚本中设置环境变量 `LC_ALL`，但不能在其调用的脚本中使用该环境变量，例如有如下脚本 *t1.sh*：
```shell
#!/bin/sh
ABC=20
echo 1:$ABC
./t2.sh
export ABC
./t2.sh
```
以及被调用脚本 *t2.sh*：
```shell
#!/bin/sh
echo "t2.sh: ABC = $ABC"
```
执行结果为：
```shell
$ ./t1.sh
1:20
t2.sh: ABC = 
t2.sh: ABC = 20
```
可以看到在执行 `export` 之前，*t2.sh* 中不能使用环境变量 ABC。

### 为变量设置默认值

```shell
#!/bin/sh
value=${value:-123}
echo $value
```
执行结果为：
```shell
$ ./t1.sh         
123
$ export value=222; ./t1.sh 
222
```

### test 命令

例如，下面的脚本测试 *A* 文件夹是否存在，如果不存在就创建该文件夹。
```shell
#!/bin/sh
test -d A || mkdir A
```

*test* 命令参数：
- `-d file`: file 是一个已存在的文件夹；
- `-z string`: string 为空；
- `-n string`: string 非空；

### tr 命令

- 替换功能，例如 `echo abc | tr abz XBZ` 的输出为 `XBc`。

### 引入其他文件执行
```shell
. auto/options
. auto/init
. auto/sources
```
`.` 等价于 `source` 命令，执行 *auto/options* 等文件中的内容，并且配置过的内容能在本脚中使用。将 *t1.sh* 修改为：
```shell
#!/bin/sh
. init
echo $XXX
./t2.sh
```
其中 *init* 文件中的内容为：
```shell
XXX=111
export ABC=22
```
执行结果为：
```shell
$ ./t1.sh
111
t2.sh: ABC = 22
```

还可以在执行 `source 命令前设置参数，例如 *t1.sh* 内容为：
```shell
#!/bin/sh
A=11 B=12 . init
```
*init* 内容为：
```shell
echo $A
echo $B
```
执行结果为：
```shell
$ ./t1.sh
11
12
```

### 检测系统类型

- `uname -s`: 检测系统名，如Linux、Darwin；
- `uname -r`: 检测版本号；
- `uname -m`: 检测硬件类型，如 x86_64

### for A do 语句

该语句用于取出用户的脚本参数，例如 *t1.sh* 的内容为：
```shell
#!/bin/sh
for opti do
    echo $opti
done
```
执行结果为：
```shell
 ./t1.sh --abc=12 -def=22 c
--abc=12
-def=22
c
```
*options* 文件中的 `for do` 循环用于取出执行 *configure* 的传入参数。

其实 `for do` 命令可以用于所有的含空白符的字符串：
```shell
#!/bin/sh
A="aa  BBB cd_f y"
for v in $A; do
    echo v=$v
done
```
执行结果为：
```shell
$ ./t1.sh
v=aa
v=BBB
v=cd_f
v=y
```

### 将含多个值的选项用单引号括起来

*t1.sh* 内容为：
```shell
#!/bin/sh
for option do
    echo $option | sed -e "s/\(--[^=]*=\)\(.* .*\)/\1'\2'/"
done
```
执行结果为：
```shell
$ ./t1.sh --abc=12 "--xx=ab cd" "-y=ab  cd" 
--abc=12
--xx='ab cd'
-y=ab cd
```

#### sed -n
`-n` 选项只打印模式匹配的行，通常与 `p` 一起使用：
```shell
#!/bin/sh
for option do
    echo $option | sed -n -e "s/\(--[^=]*=\)\(.* .*\)/\1'\2'/p"
done
```
执行结果为：
```shell
$  ./t1.sh --abc=12 "--xx=ab cd" "-y=ab  cd" 
--xx='ab cd'
```

### switch case

*t1.sh* 内容为：
```shell
#!/bin/sh
for option do
    case "$option" in
        -*=*)   value=`echo "$option" | sed -e 's/[-_a-zA-Z0-9]*=//'`;;
        *)      value="";;
    esac
    case "$option" in
        A)          echo "Get A";;
        --prefix=*) echo "O=$value";;
        -apple=*)   echo "Apple=$value";;
        B | C)        echo "B or C";;
        *)          echo "Others";
    esac
done
```
执行结果为：
```shell
$ ./t1.sh -apple=12 A B --prefix=fff C -xyz
Apple=12
Get A
B or C
O=fff
B or C
Others
```
*auto/options* 中的类似代码用于根据用户参数配置环境变量。

### if...then

```shell
#!/bin/sh
if [ $help = yes ]; then
    echo YES
else
    echo NO
fi
```
执行结果为：
```shell
$ export help=yes; ./t1.sh 
YES
$ export help=y; ./t1.sh 
NO
```

#### if [-x file]

用于判断文件存在且是可执行的。
```shell
#!/bin/sh
if [ -x tmp ]; then
    echo "tmp exist"
else
    echo "no tmp"
fi
```
执行结果为：
```shell
$ ./t1.sh 
no tmp
$ touch tmp; ./t1.sh
no tmp
$ chmod u+x tmp; ./t1.sh
tmp exist
```

#### if command

if 也可以用于判断程序是否执行成功。
```shell
#!/bin/sh
if /bin/sh -c "exit 1"; then
    echo "1"
else
    echo "2"
fi
```
执行输出为 `2`，如果改为 `exit 0`，输出为1。即执行成功，进入 if 分支，否则进入 else 分支。

#### 测试输出是否有指定字符串

```shell
#!/bin/sh
if cat tmp 2>&1 | grep 'apple' >/dev/null 2>&1; then
    echo "apple in tmp"
else
    echo "apple not in tmp"
fi
```
执行结果为：
```shell
$ echo "abc\n xappley\n xyzx" > tmp; ./t1.sh 
apple in tmp
$ echo "xxx\nyyy" > tmp; ./t1.sh 
apple not in tmp
```

### cat END

END 之间的内容可以保持原样输出。
```shell
#!/bin/sh
cat << END
    aaa
    
    bbb
    c
    ddd
END
```
执行结果为：
```shell
$ ./t1.sh
    aaa
    
    bbb
    c
    ddd
```
也可以将内容写入指定文件：
```shell
#!/bin/sh
cat << END > tmp
    aaa
    
    bbb
    c
    ddd
END
```
执行结果为：
```shell
$ ./t1.sh
$ cat tmp
    aaa
    
    bbb
    c
    ddd
```

