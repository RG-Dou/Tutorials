

# Shell "If" tutorial

### If 的基本语法

```shell
if [ command ];then
  符合该条件执行的语句
elif [ command ];then
  符合该条件执行的语句
else
  符合该条件执行的语句
fi
```



### **文件/文件夹(目录)判断**

```shell
[ -b FILE ] 如果 FILE 存在且是一个块特殊文件则为真。
[ -c FILE ] 如果 FILE 存在且是一个字特殊文件则为真。
[ -d DIR ] 如果 FILE 存在且是一个目录则为真。
[ -e FILE ] 如果 FILE 存在则为真。
[ -f FILE ] 如果 FILE 存在且是一个普通文件则为真。
[ -g FILE ] 如果 FILE 存在且已经设置了SGID则为真。
[ -k FILE ] 如果 FILE 存在且已经设置了粘制位则为真。
[ -p FILE ] 如果 FILE 存在且是一个名字管道(F如果O)则为真。
[ -r FILE ] 如果 FILE 存在且是可读的则为真。
[ -s FILE ] 如果 FILE 存在且大小不为0则为真。
[ -t FD ] 如果文件描述符 FD 打开且指向一个终端则为真。
[ -u FILE ] 如果 FILE 存在且设置了SUID (set user ID)则为真。
[ -w FILE ] 如果 FILE存在且是可写的则为真。
[ -x FILE ] 如果 FILE 存在且是可执行的则为真。
[ -O FILE ] 如果 FILE 存在且属有效用户ID则为真。
[ -G FILE ] 如果 FILE 存在且属有效用户组则为真。
[ -L FILE ] 如果 FILE 存在且是一个符号连接则为真。
[ -N FILE ] 如果 FILE 存在 and has been mod如果ied since it was last read则为真。
[ -S FILE ] 如果 FILE 存在且是一个套接字则为真。
[ FILE1 -nt FILE2 ] 如果 FILE1 has been changed more recently than FILE2, or 如果 FILE1 exists and FILE2 does not则为真。
[ FILE1 -ot FILE2 ] 如果 FILE1 比 FILE2 要老, 或者 FILE2 存在且 FILE1 不存在则为真。
[ FILE1 -ef FILE2 ] 如果 FILE1 和 FILE2 指向相同的设备和节点号则为真。
```



### **字符串判断**

```shell
[ -z STRING ] 如果STRING的长度为零则为真 ，即判断是否为空，空即是真；
[ -n STRING ] 如果STRING的长度非零则为真 ，即判断是否为非空，非空即是真；
[ STRING1 = STRING2 ] 如果两个字符串相同则为真 ；
[ STRING1 != STRING2 ] 如果字符串不相同则为真 ；
[ STRING1 ]　 如果字符串不为空则为真,与-n类似
```



### **数值判断**

```shell
INT1 -eq INT2           INT1和INT2两数相等为真 ,=
INT1 -ne INT2           INT1和INT2两数不等为真 ,<>
INT1 -gt INT2            INT1大于INT1为真 ,>
INT1 -ge INT2           INT1大于等于INT2为真,>=
INT1 -lt INT2             INT1小于INT2为真 ,<
INT1 -le INT2             INT1小于等于INT2为真,<=
```



### **复杂逻辑判断**

```shell
-a 与
-o 或
! 非

exp1: 如果a>b且a
if (( a > b )) && (( a < c ))
或者
if [[ $a > $b ]] && [[ $a < $c ]]
或者
if [ $a -gt $b -a $a -lt $c ]

exp2:如果a>b或a
if (( a > b )) || (( a < c ))
或者
if [[ $a > $b ]] || [[ $a < $c ]]
或者
if [ $a -gt $b -o $a -lt $c ]

"||"和"&&"在SHELL里可以用，也就是第一个写成if [ a>b && a
```



### [] [[]] 和test的区别

#### []和test

两者是一样的，在命令行里test expr和[ expr ]的效果相同。

test的三个基本作用是判断文件、判断字符串、判断整数。支持使用 ”与或非“ 将表达式连接起来。

test中可用的比较运算符只有==和!=，两者都是用于字符串比较的，不可用于整数比较，整数比较只能使用-eq, -gt这种形式。

无论是字符串比较还是整数比较都千万不要使用大于号小于号。当然，如果你实在想用也是可以的，对于字符串比较可以使用尖括号的转义形式， 如果比较"ab"和"bc"：[ ab \< bc ]，结果为真，也就是返回状态为0.

#### [[ ]]

这是内置在shell中的一个命令，它就比刚才说的test强大的多了。支持字符串的模式匹配（使用=~操作符时甚至支持shell的正则表达 式）。逻辑组合可以不使用test的-a,-o而使用&& ||。
字符串比较时可以把右边的作为一个模式（这是右边的字符串不加双引号的情况下。如果右边的字符串加了双引号，则认为是一个文本字符串。），而不仅仅是一个字符串，比如[[ hello == hell? ]]，结果为真。

　　注意：使用[]和[[]]的时候不要吝啬空格，每一项两边都要有空格，[[ 1 == 2 ]]的结果为“假”，但[[ 1==2 ]]的结果为“真”！



#### let和(())

两者也是一样的(或者说基本上是一样的，双括号比let稍弱一些)。主要进行算术运算(上面的两个都不行)，也比较适合进 行整数比较，可以直接使用熟悉的<,>等比较运算符。可以直接使用变量名如var而不需要$var这样的形式。支持分号隔开的多个表达式