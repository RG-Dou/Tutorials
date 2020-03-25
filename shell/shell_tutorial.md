# Shell Basic Tutorial

在用户登录到Linux系统后，系统将启动一个用户shell。我们后续所有的shell都是该shell的子shell。因此shell脚本中的命令也就是Linux Command。只不过shell脚本中可以定义变量。



### 引号的应用

【`】

 学名叫“倒引号”， 如果被“倒引号”括起来， 表示里面需要执行的是命令。
比如 `dirname $0`， 就表示需要执行  dirname $0 这个命令

【“”】 ， 被双引号括起来的内容， 里面 出现 $ (美元号： 表示取变量名) `（倒引号： 表示执行命令）  \（转义号： 表示转义），  其余的才表示字符串。
【’‘】， 被单引号括起来的内容， 里面所有的都表示串， 包括上面所说的 三个特殊字符。



### 变量和eport

变量只能在该shell脚本中使用。export 定义的变量可以被其他子shell引用。

export命令将使系统在创建每一个新的子shell时，定义这个变量的一个拷贝

```shell
export [-fnp][变量名称]=[变量设置值]
```

- -f 　代表[变量名称]中为函数名称。
- -n 　删除指定的变量。变量实际上并未删除，只是不会输出到后续指令的执行环境中。
- -p 　列出所有的shell赋予程序的环境变量。



### dirname and pwd

`dirname + 文件名`： 返回该文件的相对目录

`dirname $0`： 返回该shell脚本的相对目录

`pwd`： Linux 的一个命令，返回当前你所在的绝对目录



因此： 

```shell
cd `dirname $0`
echo `pwd`
```

这两句的目的是返回shell脚本的绝对目录，不管你是在哪里执行的该shell脚本



### BASH_SOURCE

BASH_SOURCE表示的是用户所在的目录到脚本的路径。例如测试脚本如下：

```shell
[root@hadoop01 sbin]# ./test 
./test
[root@hadoop01 sbin]# cd ..
[root@hadoop01 hadoop-2.7.7]# sbin/test 
sbin/test
```

${BASH_SOURCE-$0} ：获取当前执行的脚本文件的全路径

$0 和 $BASH_SOURCE的区别：当脚本是用./test.sh 或者 sh test.sh执行的话，$0返回的是文件目录，但是如果用source test.sh，$0返回的就是-bash，但$BASH_SOURCE就会返回当前文件目录。



### If 的使用

见shell_if_tutorial.md



### fork, source 和 exec执行shell脚本

fork: 当前进程创建一个子进程

```shell
./mytest.sh
或者
bash mytest.sh
```

source：不另外创建子进程，而是在当前的的Shell环境中执行

```shell
source ./mytest.sh
或者 （source与".”等价）
. ./mytest.sh
```

exec：不另外创建子进程，但是会终止当前的shell执行（其实我觉得这样理解可能更准确：使用exec会在当前的进程空间创建一个子线程，然后终止当前线程的执行，到了新建的线程执行完之后，其实两个线程都终止了，也就是这个当前shell进程也就终止了

```shell
exec ./mytest.sh
```



### $

$0 这个脚本/程序的执行名字
$n 这个脚本/程序的第n个参数值，n=1..9
$* 这个脚本/程序的所有参数,此选项参数可超过9个。
$# 这个脚本/程序的参数个数
$$ 这个脚本/程序的PID(脚本运行的当前[进程ID](https://www.baidu.com/s?wd=进程ID&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)号)
$! 执行上一个背景指令的PID(后台运行的最后一个进程的[进程ID](https://www.baidu.com/s?wd=进程ID&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)号)
$? 执行上一个指令的返回值 (显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误)
$- 显示shell使用的当前选项，与set命令功能相同
$@ 跟$*类似，但是可以当作数组用



$() 是将括号内命令的执行结果赋值给变量，有点像``

${} 用来作变量替换。一般情况下，$var 与 ${var} 并没有啥不一样

${} 的特殊功能：

```shell
假设我们定义了一个变量为：file=/dir1/dir2/dir3/my.file.txt
${file#*/}：拿掉第一条 / 及其左边的字符串：dir1/dir2/dir3/my.file.txt
${file##*/}：拿掉最后一条 / 及其左边的字符串：my.file.txt
${file#*.}：拿掉第一个 . 及其左边的字符串：file.txt
${file##*.}：拿掉最后一个 . 及其左边的字符串：txt
${file%/*}：拿掉最后条 / 及其右边的字符串：/dir1/dir2/dir3
${file%%/*}：拿掉第一条 / 及其右边的字符串：(空值)
${file%.*}：拿掉最后一个 . 及其右边的字符串：/dir1/dir2/dir3/my.file
${file%%.*}：拿掉第一个 . 及其右边的字符串：/dir1/dir2/dir3/my

记忆的方法为：
# 是去掉左边(在鉴盘上 # 在 $ 之左边)
% 是去掉右边(在鉴盘上 % 在 $ 之右边)
单一符号是最小匹配﹔两个符号是最大匹配。
${file#/}（不加*号）表示只去掉最左边的/

${file:0:5}：提取最左边的 5 个字节：/dir1
${file:5:5}：提取第 5 个字节右边的连续 5 个字节：/dir2

我们也可以对变量值里的字符串作替换：
${file/dir/path}：将第一个 dir 提换为 path：/path1/dir2/dir3/my.file.txt
${file//dir/path}：将全部 dir 提换为 path：/path1/path2/path3/my.file.txt

利用 ${ } 还可针对不同的变量状态赋值(没设定、空值、非空值)： 
${file-my.file.txt} ：假如 $file 没有设定，则使用 my.file.txt 作传回值。(空值及非空值时不作处理) 
${file:-my.file.txt} ：假如 $file 没有设定或为空值，则使用 my.file.txt 作传回值。 (非空值时不作处理)
${file+my.file.txt} ：假如 $file 设为空值或非空值，均使用 my.file.txt 作传回值。(没设定时不作处理)
${file:+my.file.txt} ：若 $file 为非空值，则使用 my.file.txt 作传回值。 (没设定及空值时不作处理)
${file=my.file.txt} ：若 $file 没设定，则使用 my.file.txt 作传回值，同时将 $file 赋值为 my.file.txt 。 (空值及非空值时不作处理)
${file:=my.file.txt} ：若 $file 没设定或为空值，则使用 my.file.txt 作传回值，同时将 $file 赋值为 my.file.txt 。 (非空值时不作处理)
${file?my.file.txt} ：若 $file 没设定，则将 my.file.txt 输出至 STDERR。 (空值及非空值时不作处理)
${file:?my.file.txt} ：若 $file 没设定或为空值，则将 my.file.txt 输出至 STDERR。 (非空值时不作处理)
${#file} 可得到 27 ，因为 /dir1/dir2/dir3/my.file.txt 刚好是 27 个字节
```

