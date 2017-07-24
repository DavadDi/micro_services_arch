# Systemtap快速入门

## 1. 介绍

### 1.1 Hello World

Systemtap 为开发者或者管理者提供了一种深度检测运行 Linux 操作系统的能力，使用者通过编写一定的简单脚本，就可以获取内核级别的调试信息。

event  —>  handlers， 使用者通过对于特定 event 定义一个或者一组 handler(s), 当 event 触发的时候，handler(s) 被调用。event 可以为 函数调用（进入或者退出）、 定时器到期、system tap 会话开始或者结束。 

systemtap 的工作机制是通过将用户编写的脚本程序转化成 c 代码，将 c 代码编译成内核模块，挂载到系统内部的方式来进行数据的获取。



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

监控系统调用 open 
```shell
# cat strace-open.stp

probe syscall.open
{
	printf("%s(%d) open (%s)\n", execname(), pid(), argstr)  
}

probe timer.s(4)
{
  exit()
}

#stap strace-open.stp
redis-server(1511) open ("/proc/1511/stat", O_RDONLY)
vmtoolsd(576) open ("/proc/meminfo", O_RDONLY)
......
```



```shell
# stap -L "syscall.*"
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

### 1.2 Probe Points

Probe Points:

| `begin`                                  | The startup of the systemtap session.    |
| ---------------------------------------- | ---------------------------------------- |
| `end`                                    | The end of the systemtap session.        |
| `kernel.function("sys_open")`            | The entry to the function named `sys_open` in the kernel. |
| `syscall.close.return`                   | The return from the `close` system call. |
| `module("ext3").statement(0xdeadbeef)`   | The addressed instruction in the `ext3` filesystem driver. |
| `timer.ms(200)`                          | A timer that fires every 200 milliseconds. |
| `timer.profile`                          | A timer that fires periodically on every CPU. |
| `perf.hw.cache_misses`                   | A particular number of CPU cache misses have occurred. |
| `procfs("status").read`                  | A process trying to read a synthetic file. |
| `process("a.out").statement("*@main.c:200")` | Line 200 of the `a.out` program.         |
| `kernel.function("*@net/socket.c").call` | Any function defined in   `net/socket.c` |



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



突破systemtap定义变量的限制：
$ sudo stap -DMAXMAPENTRIES=10240 test.stp

#### Links:####

1. [System tap 官方文档](https://sourceware.org/systemtap/documentation.html)  
   - [Systemtap Tutorial](https://sourceware.org/systemtap/tutorial/tutorial.html)  
   - [SystemTap_Beginners_Guide](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SystemTap_Beginners_Guide/introduction.html) 
   - [SystemTap Language Reference](https://sourceware.org/systemtap/langref/)
   - [SystemTap Tapset Reference Manual](https://sourceware.org/systemtap/tapsets/index.html)
2. [System Best Examples](https://sourceware.org/systemtap/examples/)  [TCP_Trace.stp](https://sourceware.org/systemtap/examples/network/tcp_trace.stp)
3. [SYSTEMTAP BEGINNERS GUIDE]([https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SystemTap_Beginners_Guide/index.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SystemTap_Beginners_Guide/index.html))
4. [CentOS7安装systemtap](http://www.hi-roy.com/2016/07/27/CentOS7%E5%AE%89%E8%A3%85systemtap/)
5. [openresty-systemtap-toolkit](https://github.com/openresty/openresty-systemtap-toolkit/)
6. [Systemtap笔记](http://nanxiao.me/tag/systemtap/)
7. [Linux 内核的tcp连接状态分析](http://gmd20.blog.163.com/blog/static/168439232014741166246/)
8. [Systemtap examples, Network - 4 Monitoring TCP Packets](http://www.cnblogs.com/zengkefu/p/6372276.html)
9. [systemtap的网络监控脚本](http://m.blog.itpub.net/15480802/viewspace-762002/)
10. [Go 火焰图](http://lihaoquan.me/2017/1/1/Profiling-and-Optimizing-Go-using-go-torch.html)

[systemtap变量范围限制]: http://blog.yufeng.info/archives/1213&quot;突破systemtap脚本对资源使用的限制&quot;
```

```




