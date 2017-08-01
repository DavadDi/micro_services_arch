# Systemtap快速入门



![Trace](http://www.brendangregg.com/perf_events/perf_events_map.png)

## 1. 介绍

### 1.1 Hello World

Systemtap 为开发者或者管理者提供了一种深度检测运行 Linux 操作系统的能力，使用者通过编写一定的简单脚本，就可以获取内核级别的调试信息。

systemtap 脚本中基本元素是 `event`, 用户为特定 `event` 编写相对应的 `handler` 进行相关动作处理。

 `event` 可能包括以下类型：

- 调用函数或者调用函数结束
- 定时器触发
- systemtap 会话开始或者结束

systemtap 的工作机制是通过将用户编写的脚本程序转化成 c 代码，将 c 代码编译成内核模块，挂载到系统内部的方式来进行数据的获取。

![](http://hi.csdn.net/attachment/201104/23/0_1303533510e2Tx.gif)

以 `root` 用户运行或者以 `stapdev` group 中具备`sudo` 权限的用户运行。

```bash
# cat hello-world.stp

probe begin
{
  print("hello world!\n")
  exit()
}

# stp hello-world.stp
hello world!
```

监控系统调用 syscall.open 
```shell
# cat strace-open.stp

probe syscall.open
{
    # stap -e script -c cmd -x pid  
	# stap  -vv -e 'probe syscall.open {printf("name: %s args: %s [var: %s parms: %s]\n", name, argstr, $$vars, $$parms)}'

	printf("%s(%d) open (%s)\n", execname(), pid(), argstr)  
}

probe timer.s(4)
{
  exit()
}

# stap strace-open.stp
redis-server(1511) open ("/proc/1511/stat", O_RDONLY)
vmtoolsd(576) open ("/proc/meminfo", O_RDONLY)
......
```



```shell
# stap -L "syscall.*"
# stap -L 'kernel.function("inet_csk_accept")'
# stap -L 'kernel.function("inet_csk_accept").return'
# stap -L "syscall.open"
syscall.open filename:string mode:long __nr:long name:string flags:long argstr:string $filename:long int $flags:long int $mode:long int

> filename:string  open file name
> mode:long        open file mode
> __nr:long        number of params ??
> flags:long       open file flags

> name:string      syscall name: open here
> argstr:string    alias for print 

# 对于syscall 还定义了一些 aliases, 主要是用于调测：
# argstr: A pretty-printed form of the entire argument list, without parentheses.
# name: The name of the system call.
# retstr: For return probes, a pretty-printed form of the system call result.
# $$parms  filename=0x7fff3f2cf330 flags=0x0 mode=0x7fff3f2cf33f

# stap -e 'probe syscall.open{printf("argstr is %s, __nr is %d\n", argstr, __nr)}'
# stap -e 'probe syscall.open{printf("filename is %s, name is %s, flags is 0x%x, mode is 0x%x\n", filename, name, flags, mode)}'
# stap -e 'probe syscall.open{printf("filename is 0x%x, $flags is 0x%x, $mode is 0x%x\n", $filename, $flags, $mode)}'
```

#### stap 命令行参数

- -v [vvv]

  输出更加详细的信息，v越多信息越详细

- `-o filename`

  将屏幕输出到指定文件中，可以结合 `-S` 控制日志的大小

- `-S size,count`

  用于控制输出文件的大小和数目，`size` 单位是 `M`  ，日志文件后面会包含一个序号的后缀

- `-x process ID`

  设定脚本中 `target()` 的值， 主要用于监控特定进程的事件类型

  ```shell
  # stap  -x 1585 -vv -e 'probe syscall.open {if (target() == pid()) {printf("name: %s args: %s\n", name, a
  ```

- `-c command`

  运行指定的命令，同时设定了 `target()` 函数指向了运行命令的 `pid`,  运行命令必须是全路径

  ```shell
  # stap  -vv -e 'probe syscall.open {printf("name: %s args: %s\n", name, argstr)}' -c "/bin/ls"
  ```

- `-e 'script'`

  ```shell
  # stap -e 'probe begin{ println("Hello"); exit() }'
  ```

- `-F`

  运行后台模式，可以随时切换过去

`stap` can also be instructed to run scripts from the standard input using the `-` switch. To illustrate:

```shell
# echo "probe timer.s(1) {exit()}" | stap -
```

整体架构：

![](http://hi.csdn.net/attachment/201104/23/0_1303533747D3xY.gif)

流程：

![](http://officialblog-wordpress.stor.sinaapp.com/uploads/2013/11/figure2.gif)

### 1.2 Probe Points

The library of scripts that comes with systemtap, each called a `tapset`, `man stapprobes`获取更详细的帮助。

```shell
probe PROBEPOINT [, PROBEPOINT] { [STMT ...] }
```

Probe: 

- synchronous  event(contextual data)
- asynchronous timer

Probe Points:

| `begin`                                  | The startup of the systemtap session.    |
| ---------------------------------------- | ---------------------------------------- |
| `end`                                    | The end of the systemtap session.        |
| `error`                                  | The *error* probe point is similar to the *end* probe, except that each such probe handler run when the session ends after errors have occurred. In such cases, "end" probes are skipped, but each "error" probe is still attempted. This kind of probe can be used to clean up or emit a "final gasp". It may also be numerically parametrized to set a sequence. |
| `never`                                  | Its probe handler is never run, though its statements are analyzed for symbol / type correctness as usual. This probe point may be useful in conjunction with optional probes. |
| `kernel.function("sys_open")`            | The entry to the function named `sys_open` in the kernel. |
| `syscall.close.return`                   | The return from the `close` system call. |
| `module("ext3").statement(0xdeadbeef)`   | The addressed instruction in the `ext3` filesystem driver. |
| `timer.ms(200)`                          | A timer that fires every 200 milliseconds. |
| `timer.profile`                          | A timer that fires periodically on every CPU. |
| `perf.hw.cache_misses`                   | A particular number of CPU cache misses have occurred. |
| `procfs("status").read`                  | A process trying to read a synthetic file. |
| `process("a.out").statement("*@main.c:200")` | Line 200 of the `a.out` program.         |
| `kernel.function("*@net/socket.c").call` | Any function defined in   `net/socket.c` |



> probe points 可以使用 `aliases`或者`suffix`, 例如 `syscall.read.return.maxactive(10)` == `kernel.function("sys_read").return.maxactive(10)`；probe points 后面可以添加 `?` 表示可选，如果发生展开错误则忽略。

> However, a probe point may be followed by a "?" character, to indicate that it is optional, and that no error should result if it fails to resolve. Optionalness passes down through all levels of alias/wildcard expansion. Alternately, a probe point may be followed by a "!" character, to indicate that it is both optional and sufficient. (Think vaguely of the Prolog cut operator.) If it does resolve, then no further probe points in the same comma-separated list will be resolved. Therefore, the "!" sufficiency mark only makes sense in a list of probe point alternatives. [man](https://sourceware.org/systemtap/man/stapprobes.3stap.html)



| **DWARF**                    | NON-DWARF                 | SYMBOL-TABLE        |
| ---------------------------- | ------------------------- | ------------------- |
| kernel.function, .statement  | kernel.mark               | kernel.function***  |
| module.function, .statement  | process.mark, process.plt | module.function***  |
| process.function, .statement | begin, end, error, never  | process.function*** |
| process.mark***              | timer                     |                     |
| .function.callee             | perf                      |                     |
|                              | procfs                    |                     |

| **AUTO-GENERATED-DWARF** | kernel.statement.absolute  |      |
| ------------------------ | -------------------------- | ---- |
|                          | kernel.data                |      |
| kernel.trace             | kprobe.function            |      |
|                          | process.statement.absolute |      |
|                          | process.begin, .end        |      |
|                          | netfilter                  |      |
|                          | java                       |      |

Here is a list of DWARF probe points currently supported:

```
kernel.function(PATTERN)
kernel.function(PATTERN).call
kernel.function(PATTERN).callee(PATTERN)
kernel.function(PATTERN).callee(PATTERN).return
kernel.function(PATTERN).callee(PATTERN).call
kernel.function(PATTERN).callees(DEPTH)
kernel.function(PATTERN).return
kernel.function(PATTERN).inline
kernel.function(PATTERN).label(LPATTERN)
module(MPATTERN).function(PATTERN)
module(MPATTERN).function(PATTERN).call
module(MPATTERN).function(PATTERN).callee(PATTERN)
module(MPATTERN).function(PATTERN).callee(PATTERN).return
module(MPATTERN).function(PATTERN).callee(PATTERN).call
module(MPATTERN).function(PATTERN).callees(DEPTH)
module(MPATTERN).function(PATTERN).return
module(MPATTERN).function(PATTERN).inline
module(MPATTERN).function(PATTERN).label(LPATTERN)
kernel.statement(PATTERN)
kernel.statement(PATTERN).nearest
kernel.statement(ADDRESS).absolute
module(MPATTERN).statement(PATTERN)
process("PATH").function("NAME")
process("PATH").statement("*@FILE.c:123")
process("PATH").library("PATH").function("NAME")
process("PATH").library("PATH").statement("*@FILE.c:123")
process("PATH").library("PATH").statement("*@FILE.c:123").nearest
process("PATH").function("*").return
process("PATH").function("myfun").label("foo")
process("PATH").function("foo").callee("bar")
process("PATH").function("foo").callee("bar").return
process("PATH").function("foo").callee("bar").call
process("PATH").function("foo").callees(DEPTH)
process(PID).function("NAME")
process(PID).function("myfun").label("foo")
process(PID).plt("NAME")
process(PID).plt("NAME").return
process(PID).statement("*@FILE.c:123")
process(PID).statement("*@FILE.c:123").nearest
process(PID).statement(ADDRESS).absolute
```

> SYSCALL and ND_SYSCALL, Generally, a pair of probes are defined for each normal system call as listed in the syscalls(2) manual page, one for entry and one for return. Those system calls that never return do not have a corresponding .return probe. The nd_* family of probes are about the same, except it uses non-DWARF based searching mechanisms, which may result in a lower quality of symbolic context data (parameters), and may miss some system calls. You may want to try them first, in case kernel debugging information is not immediately available.
>
> ND_SYSCALL **non-DWARF**， 主要是基于没有 **DWARF** 调试信息的情况下，采用更加底层的符号来处理。
>
> 对于系统调用，除了调用函数的参数，还有以下通用参数可以使用：
>
> **argstr**
>
> A pretty-printed form of the entire argument list, without parentheses.
>
> **name**
>
> The name of the system call.
>
> **retstr**
>
> For return probes, a pretty-printed form of the system-call result.



#### [CONTEXT VARIABLES](https://sourceware.org/systemtap/man/stapprobes.3stap.html)

- $var

  refers to an in-scope variable "var". If it's an integer-like type, it will be cast to a 64-bit int for systemtap script use. String-like pointers (char *) may be copied to systemtap string values using the *kernel_string* or *user_string* functions.

- @var("varname")

  an alternative syntax for *$varname*

- @var("varname@src/file.c")

  refers to the global (either file local or external) variable *varname* defined when the file *src/file.c* was compiled. The CU in which the variable is resolved is the first CU in the module of the probe point which matches the given file name at the end and has the shortest file name path (e.g. given *@var(foo@bar/baz.c)* and CUs with file name paths *src/sub/module/bar/baz.c* and *src/bar/baz.c* the second CU will be chosen to resolve the (file) global variable *foo*

- $var->field traversal via a structure's or a pointer's field. This

  generalized indirection operator may be repeated to follow more levels. Note that the *.* operator is not used for plain structure members, only*->* for both purposes. (This is because "." is reserved for string concatenation.)

- $return

  is available in return probes only for functions that are declared with a return value, which can be determined using @defined($return).

- $var[N]

  indexes into an array. The index given with a literal number or even an arbitrary numeric expression.

A number of operators exist for such basic context variable expressions:

- $$vars

  expands to a character string that is equivalent to`sprintf("parm1=%x ... parmN=%x var1=%x ... varN=%x",        parm1, ..., parmN, var1, ..., varN)`for each variable in scope at the probe point. Some values may be printed as *=?* if their run-time location cannot be found.

- $$locals

  expands to a subset of $$vars for only local variables.

- $$parms

  expands to a subset of $$vars for only function parameters.

- $$return

  is available in return probes only. It expands to a string that is equivalent to sprintf("return=%x", $return) if the probed function has a return value, or else an empty string.

- & $EXPR

  expands to the address of the given context variable expression, if it is addressable.

- @defined($EXPR)

  expands to 1 or 0 iff the given context variable expression is resolvable, for use in conditionals such as`@defined($foo->bar) ? $foo->bar : 0`

- $EXPR$

  expands to a string with all of $EXPR's members, equivalent to`sprintf("{.a=%i, .b=%u, .c={...}, .d=[...]}",         $EXPR->a, $EXPR->b)`

- $EXPR$$

  expands to a string with all of $var's members and submembers, equivalent to`sprintf("{.a=%i, .b=%u, .c={.x=%p, .y=%c}, .d=[%i, ...]}",        $EXPR->a, $EXPR->b, $EXPR->c->x, $EXPR->c->y, $EXPR->d[0])`



return:

In addition, arbitrary entry-time expressions can also be saved for ".return" probes using the *@entry(expr)* operator. For example, one can compute the elapsed time of a function:

```
probe kernel.function("do_filp_open").return {    println( get_timeofday_us() - @entry(get_timeofday_us()) )}
```



### 1.3 Print info

| `tid()`              | The id of the current thread.            |
| -------------------- | ---------------------------------------- |
| `pid()`              | The process (task group) id of the current thread. |
| `uid()`              | The id of the current user.              |
| `execname()`         | The name of the current process.         |
| `cpu()`              | The current cpu number.                  |
| `gettimeofday_s()`   | Number of seconds since epoch.           |
| `get_cycles()`       | Snapshot of hardware cycle counter.      |
| `pp()`               | A string describing the probe point being currently handled. |
| `ppfunc()`           | If known, the the function name in which this probe was placed. |
| `$$vars`             | If available, a pretty-printed listing of all local variables in scope. |
| `print_backtrace()`  | If possible, print a kernel backtrace.   |
| `print_ubacktrace()` | If possible, print a user-space backtrace. |
| thread_indent()      | Given an indentation delta parameter, it stores internally an indentation counter for each thread (`tid()`), and returns a string with some generic trace data plus an appropriate number of indentation spaces. |



```shell
# cat thread_indent.stp

probe kernel.function("*@net/socket.c") 
{
  printf ("%s -> %s\n", thread_indent(1), probefunc())
}
probe kernel.function("*@net/socket.c").return 
{
  printf ("%s <- %s\n", thread_indent(-1), probefunc())
}
```



## 2. 安装

kernel development tools and debugging data

[CentOS7安装systemtap](http://www.hi-roy.com/2016/07/27/CentOS7%E5%AE%89%E8%A3%85systemtap/)


```shell
# yum install systemtap systemtap-runtime
# yum install kernel-devel-`uname -r` kernel-debuginfo-common-`uname -r` kernel-debuginfo-`uname -r`
```



## 3. Analysis

### 3.1 Basic constructs

```
if (EXPR) STATEMENT [else STATEMENT]

while (EXPR) STATEMENT

for (A; B; C) STATEMENT
```

注释：

> \# this is a commnet
> /* this is a commnet */
> // this is a commnet

```
if (uid() > 100)
if (execname() == "sed")
if (cpu() == 0 && gettimeofday_s() > 1140498000)
"hello" . " " . "world"     字符串拼接
```


| foo = gettimeofday_s()         | foo is a number             |
| ------------------------------ | --------------------------- |
| bar = "/usr/bin/" . execname() | bar is a string             |
| c++                            | c is a number               |
| s = sprint(2345)               | s becomes the string "2345" |



默认情况下，`variable` 是 local， 全局的变量需要使用 `global` 进行声明。



### 3.2 Target variables

### 3.3 Function

`Function` 定义不要求顺序；全局变量也不要求定义的顺序；

```c
function makedev(major, minor)
{
  return major << 20 | minor
}
```



### 3.4 Arrays

systemtap 中的数组实际上是 `hashtable` 结构， 必须要声明为**全局变量**。

```c
global stat        # 默认大小最大 2048 个， MAXMAPENTRIES
global stat[4096]  # 保留4096个空间
```



`Array` 常用的操作是设置变量和查找， 使用 `awk`的语法，` array_name[key] = value`,  其中 `key` 可以为多个变量的组合，最大为 9 个，中间使用 `,`分割，例如： `array_name[key1_1,key1_2] = value`

| `if ([4,"hello"] in foo) { }` | membership test              |
| ----------------------------- | ---------------------------- |
| `delete times[tid()]`         | deletion of a single element |
| `delete times`                | deletion of all elements     |



对于 Arrays，可以使用 `foreach` 来进行遍历，同时支持使用 `+`或者`-`进行排序。

| `foreach (x = [a,b] in foo) { fuss_with(x) }` | simple loop in arbitrary sequence        |
| ---------------------------------------- | ---------------------------------------- |
| `foreach ([a,b] in foo+ limit 5) { }`    | loop in increasing sequence of value, stop after 5 |
| `foreach ([a-,b] in foo) { }`            | loop in decreasing sequence of first key |



### 3.5 Aggregates

主要用于数据统计，添加操作类型为 "<<<"



```shell
a <<< delta_timestamp
writes[execname()] <<< count
```



| @avg(a)                         | the average of all the values accumulated into `a` |
| ------------------------------- | ---------------------------------------- |
| print(@hist_linear(a,0,100,10)) | print an ``ascii art'' linear histogram of the same data stream, bounds  0…100,  bucket width is 10 |
| @count(writes["zsh"])           | the number of times ``zsh'' ran the probe handler |
| print(@hist_log(writes["zsh"])) | print an ``ascii art'' logarithmic histogram of the same data stream |



### 3.6 Safety

1. systemtap 会对运行的 `prober handler` 运行时间进行限制避免死循环或者无限递归；
2. 没有动态内存分配，所有的数组、函数上下文和缓存都会在 `session` 初始化的时候一次性初始化；
3. 空指针或者0除数等危险的操作，在生成C语言代码过程中都会被严格检查；
4. 转化器后者编译器的bug，生成的 C 语言代码，可以通过 `-p3` 选型进行检测。

stp 脚本安装的目录 `/usr/share/systemtap/tapset`，可以使用 `-I` 来包含用户定义的附加目录。



```
# stap -p1 -vv -e 'probe begin{}' > /dev/null
Created temporary directory "/tmp/stapKbwUbu"
Session arch: x86_64 release: 3.10.0-514.el7.x86_64
Searched for library macro files: "/usr/share/systemtap/tapset/linux/*.stpm", found: 5, processed: 5
Searched for library macro files: "/usr/share/systemtap/tapset/*.stpm", found: 8, processed: 8
Searched: "/usr/share/systemtap/tapset/linux/x86_64/*.stp", found: 3, processed: 3
Searched: "/usr/share/systemtap/tapset/linux/*.stp", found: 71, processed: 71
Searched: "/usr/share/systemtap/tapset/x86_64/*.stp", found: 5, processed: 5
Searched: "/usr/share/systemtap/tapset/*.stp", found: 35, processed: 35
Pass 1: parsed user script and 127 library scripts using 230472virt/43356res/3236shr/40484data kb, in 320usr/20sys/311real ms.
Running rm -rf /tmp/stapKbwUbu
Spawn waitpid result (0x0): 0
Removed temporary directory "/tmp/stapKbwUbu"
```



嵌入C语言代码：

stp 文件中可以，嵌入 C 语言代码，嵌入的C语言代码，可以包含头文件和代码，使用`%{`xxx `%}` 进行区分；

```shell
function <name>:<type> ( <arg1>:<type>, ... ) %{ <C_stmts> %}
```





Using a single `$` or a double `$$` suffix provides a swallow or deep string representation of the variable data type. Using a single `$`, as in `$var$`, will provide a string that only includes the values of all basic type values of fields of the variable structure type but not any nested complex type values (which will be represented with `{...}`). Using a double `$$`, as in `@var("var")$$` will provide a string that also includes all values of nested data types.

`$$vars` expands to a character string that is equivalent to `sprintf("parm1=%x ... parmN=%x var1=%x ... varN=%x", $parm1, ..., $parmN, $var1, ..., $varN)`

`$$locals` expands to a character string that is equivalent to `sprintf("var1=%x ... varN=%x", $var1, ..., $varN)`

`$$parms` expands to a character string that is equivalent to `sprintf("parm1=%x ... parmN=%x", $parm1, ..., $parmN)`





源码目录：`/usr/src/debug/kernel-3.10.0-514.el7/linux-3.10.0-514.el7.x86_64/`

定义 `probe` 的别名，例如官方提供的函数库：` /usr/share/systemtap/tapset/linux` 提供的样例：`/usr/share/doc/systemtap-client-3.0/examples`

```
probe netdev.receive
        =  kernel.function("netif_receive_skb")
{
        dev_name = kernel_string($skb->dev->name)
        length = $skb->len
        protocol = $skb->protocol
        truesize = $skb->truesize
}

# used
probe netdev.receive
{
	ifrecv[pid(), dev_name, execname(), uid()] <<< length
}
```

 

[who_send_it.stp](https://sourceware.org/systemtap/examples/network/who_sent_it.stp)

```shell
#! /usr/bin/env stap

# Print a trace of threads sending IP packets (UDP or TCP) to a given
# destination port and/or address.  Default is unfiltered.

global the_dport = 0    # override with -G the_dport=53
global the_daddr = ""   # override with -G the_daddr=127.0.0.1

probe netfilter.ip.local_out {
    if ((the_dport == 0 || the_dport == dport) &&
        (the_daddr == "" || the_daddr == daddr))
	    printf("%s[%d] sent packet to %s:%d\n", execname(), tid(), daddr, dport)
}
```



[para-callgraph.stp](https://sourceware.org/systemtap/examples/general/para-callgraph.txt)

> thread_indent() The generic data included in the returned string includes a **timestamp** (number of microseconds since the first call to `thread_indent()` by the thread), a **process name**, and the **thread ID.** 

```shell
#
# stap para-callgraph.stp 'process("/bin/ls").function("*")' \
# 'process("/bin/ls").function("main")' -c "/bin/ls > /dev/null"

#! /usr/bin/env stap

function trace(entry_p, extra) {
  %( $# > 1 %? if (tid() in trace) %)
  printf("%s%s%s %s\n",
         thread_indent (entry_p),
         (entry_p>0?"->":"<-"),
         ppfunc (),
         extra)
}


%( $# > 1 %?
global trace
probe $2.call {
  trace[tid()] = 1
}
probe $2.return {
  delete trace[tid()]
}
%)

probe $1.call   { trace(1, $$parms) }
probe $1.return { trace(-1, $$return) }
```



https://sourceware.org/systemtap/langref/Probe_points.html 

A probe point may be followed by a question mark (?) character, to indicate that it is optional, and that no error should result if it fails to expand. This effect passes down through all levels of alias or wildcard expansion.The following is the general syntax.`kernel.function("no_such_function") ?`



**DWARF** is a debugging file format used by many compilers and debuggers to support source level debugging. It addresses the requirements of a number of procedural languages, such as C, C++, and Fortran, and is designed to be extensible to other languages. DWARF is architecture independent and applicable to any processor or operating system. It is widely used on Unix, Linux and other operating systems, as well as in stand-alone environments.



A new operator, `@entry`, is available for automatically saving an expression at entry time for use in a`.return` probe.



####Syscall probes
syscall.NAME
syscall.NAME.return

* argstr: A pretty-printed form of the entire argument list, without parentheses.
* name: The name of the system call.
* retstr: For return probes, a pretty-printed form of the system call result.


#### Special probe points
begin
end
error



Pointer typecasting:

@cast(p, "type_name"[, "module"])->member

```shell
@cast(pointer, "task_struct", "kernel")->parent->tgid

# 采用头文件方式，用 <> 分开
@cast(tv, "timeval", "<sys/time.h>")->tv_sec
@cast(task, "task_struct", "kernel<linux/sched.h>")->tgid
```



命令行参数：

$1 … $<NN> 表示传输参数是整数常量

@1 … @<NN> 表示传入的是字符串常量



```shell
# stap example.stp '5+5' mystring
probe begin { printf("%d, %s\n", $1, @2) }
```



条件编译：

```shell
%( CONDITION %? TRUE-TOKENS %)
%( CONDITION %? TRUE-TOKENS %: FALSE-TOKENS %)
```
From [tcp_trace.stp](https://sourceware.org/systemtap/examples/network/tcp_trace.stp)
```
%( kernel_v > "2.6.24" %?
probe kernel.function("tcp_set_state")
{
	sk = $sk
	new_state = $state
	TCP_CLOSE = 7
	TCP_CLOSE_WAIT = 8
	key = filter_key(sk)
	if ( key && ((new_state == TCP_CLOSE)||(new_state == TCP_CLOSE_WAIT))){
		if (state_flg && state[key]) print_close(key,new_state);
		clean_up(key);
	}
}
%)
```



宏定义：

```
@define NAME %( BODY %)
@define NAME(PARAM_1, PARAM_2, ...) %( BODY %)
```



##### tapset::json
The JSON tapset provides probes, functions, and macros to generate 
 a JSON metadata and data file. The JSON metadata file is located in 
 /proc/systemtap/MODULE/metadata.json. The JSON data file is located 
 in /proc/systemtap/MODULE/data.json. The JSON data file is updated 
 with current data every time the file is read.



```
 /proc/systemtap/MODULE/metadata.json
```

From [iotime.stp](https://sourceware.org/systemtap/examples/io/iotime.stp)

```
function timestamp:long() { return gettimeofday_us() - start }
function proc:string() { return sprintf("%d (%s)", pid(), execname()) }
probe begin { start = gettimeofday_us() }

probe syscall.open.return {
  filename = user_string($filename)
  if ($return != -1) {
    filehandles[pid(), $return] = filename
  } else {
    printf("%d %s access %s fail\n", timestamp(), proc(), filename)
  }
}
```


```shell
man -k tapset::    # man pages for tapsets
man -k probe::	   # man pages for individual probe points.
man -k function::  # man -k function:
stap -L 'kernel.trace("net*")'
```



内核tracepoint:

```shell
# yum install perf
# perf list tracepoint | head
```


**stap 运行的时候可以通过 -G 设置全局变量**

[strace.stp](https://sourceware.org/systemtap/examples/process/strace.stp)  使用 nd_syscall 设置。

监控连入tcp连接

```shell
#! /usr/bin/env stap
probe begin {
	printf("%6s %16s %6s %6s %16s\n",
			"UID", "CMD", "PID", "PORT", "IP_SOURCE")
}
probe kernel.function("tcp_accept").return?,
      kernel.function("inet_csk_accept").return? {
		sock = $return
		if (sock != 0)
		      printf("%6d %16s %6d %6d %16s\n", uid(), execname(), pid(),
			     inet_get_local_port(sock), inet_get_ip_source(sock))
}
```



```shell
cat tcp_state.stp
global tcp_state;

global filter_rip
global filter_rport

probe begin
{
	filter_rip =  ipv4_pton(@1)
	filter_rport = $2;

	init_tcp_state()
	printf("Start stat now....\n")
}

function ipv4_pton:long (addr:string)
{
	i=32;
	ip=0;
	ips=addr;
	while(strlen(byte = tokenize(ips, ".")) != 0) {
		i-=8;
		ips="";

		j=strtol(byte,10);
		ip=ip+(j<<i) // left shift the byte into the address
	}

	return ip;
}

function init_tcp_state ()
{
	tcp_state[1] = "ESTABLISHED"
	tcp_state[2] = "SYN_SENT"
	tcp_state[3] = "SYN_RECV"
	tcp_state[4] = "FIN_WAIT1"
	tcp_state[5] = "FIN_WAIT2"
	tcp_state[6] = "TIME_WAIT"
	tcp_state[7] = "CLOSE"
	tcp_state[8] = "CLOSE_WAIT"
	tcp_state[9] = "LAST_ACK"
	tcp_state[10] = "LISTEN"
	tcp_state[11] = "CLOSING"
}

function state_num2str:string ( state:long )
{
	return (state in tcp_state ? tcp_state[state] : "UNDEF")
}

function filter_pkg:long( raddr:long, rport:long)
{
	if ((raddr == filter_rip) && (rport == filter_rport))
	{
		return 1
	}

	return 0
}
function print_state(sk:long, restranmit:long)
{
	laddr = tcpmib_local_addr(sk);
	raddr = tcpmib_remote_addr(sk);
	lport = tcpmib_local_port(sk);
	rport = tcpmib_remote_port(sk);

	if ((filter_pkg(raddr, rport) == 1) && (lport != 0)) {
		new_state = @cast(sk, "sock_common")->skc_state;
		str = state_num2str(new_state);

		printf("%d %s:%d %s %d\n",
			gettimeofday_ms(),
			ip_ntop(htonl(laddr)),
			lport,
			str, restranmit);
	}
}

probe kernel.function("tcp_retransmit_skb")
{
	print_state($sk,1)
}

%( kernel_v > "2.6.24" %?
probe kernel.function("tcp_set_state")
{
	print_state($sk,0)
}
%)

```



生成火焰图：

```
# git clone https://github.com/brendangregg/FlameGraph.git
# perf record -F 99 -p pid -g -- sleep 60
# perf script > out.perf
# /opt/FlameGraph/stackcollapse-perf.pl out.perf > out.folded
# /opt/FlameGraph/flamegraph.pl out.folded > cpu.svg
```

突破systemtap定义变量的限制：
$ sudo stap -DMAXMAPENTRIES=10240 test.stp

#### Links:####

1. [System tap 官方文档](https://sourceware.org/systemtap/documentation.html)  
   - [Systemtap Tutorial](https://sourceware.org/systemtap/tutorial/tutorial.html)  
   - [SystemTap_Beginners_Guide](https://sourceware.org/systemtap/SystemTap_Beginners_Guide/) 
   - [SystemTap Language Reference](https://sourceware.org/systemtap/langref/)
   - [SystemTap Tapset Reference Manual](https://sourceware.org/systemtap/tapsets/index.html)
   - [man](https://sourceware.org/systemtap/man/)
2. [System Best Examples](https://sourceware.org/systemtap/examples/)  [TCP_Trace.stp](https://sourceware.org/systemtap/examples/network/tcp_trace.stp)
3. [Centos7 SYSTEMTAP BEGINNERS GUIDE]([https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SystemTap_Beginners_Guide/index.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SystemTap_Beginners_Guide/index.html))
4. [CentOS7安装systemtap](http://www.hi-roy.com/2016/07/27/CentOS7%E5%AE%89%E8%A3%85systemtap/)
5. [openresty-systemtap-toolkit](https://github.com/openresty/openresty-systemtap-toolkit/)
6. [Systemtap笔记](http://nanxiao.me/tag/systemtap/)
7. [Linux 内核的tcp连接状态分析](http://gmd20.blog.163.com/blog/static/168439232014741166246/)
8. [Systemtap examples, Network - 4 Monitoring TCP Packets](http://www.cnblogs.com/zengkefu/p/6372276.html)
9. [systemtap的网络监控脚本](http://m.blog.itpub.net/15480802/viewspace-762002/)
10. [Go 火焰图](http://lihaoquan.me/2017/1/1/Profiling-and-Optimizing-Go-using-go-torch.html)
11. [工欲性能调优，必先利其器（2）- 火焰图](https://pingcap.com/blog-tangliu-tool-%7C%7C-zh)
12. [OpenRestry火焰图](https://moonbingbing.gitbooks.io/openresty-best-practices/flame_graph.html)
13. Perf 相关资料 
    * [perf example](http://www.brendangregg.com/perf.html)
    * [Perf -- Linux下的系统性能调优工具，第 1 部分](https://www.ibm.com/developerworks/cn/linux/l-cn-perf1/index.html)
    * [Perf -- Linux下的系统性能调优工具，第 2 部分](https://www.ibm.com/developerworks/cn/linux/l-cn-perf2/index.html)
    * [**perf-tools**](https://github.com/brendangregg/perf-tools)
14. [动态追踪技术漫谈](https://openresty.org/posts/dynamic-tracing/)
15. [Architecture of systemtap: a Linux trace/probe tool](https://sourceware.org/systemtap/archpaper.pdf)





[systemtap变量范围限制]: http://blog.yufeng.info/archives/1213&quot;突破systemtap脚本对资源使用的限制&quot;

```

```




