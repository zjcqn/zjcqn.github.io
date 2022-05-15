---
title: Python | Python 脚本中运行shell脚本
author: 陈钱牛
date: 2020-10-24 19:33:00 +0800
categories: Implementation
tags: python
math: true
layout: post
typora-root-url: ..
banner:
  video:
  loop: true
  volume: 0.8
  start_at: 8.5
  image: https://bit.ly/3xTmdUP
  opacity: 0.618
  background: "#000"
  height: "100vh"
  min_height: "38vh"
  heading_style: "font-size: 4.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
---

> 这篇文章主要介绍了Python的subprocess模块总结,本文详细讲解了subprocess模块参数及Popen方法,然后给出了多个使用实例,需要的朋友可以参考下

subprocess意在替代其他几个老的模块或者函数，比如：``os.system`` ``os.spawn*`` ``os.popen*`` ``popen2.*`` ``commands.*``。subprocess最简单的用法就是调用shell命令了,另外也可以调用程序,并且可以通过stdout,stdin和stderr进行交互。

# API

```python
# subprocess.Popen's api
subprocess.Popen(
      args,
      bufsize=0,
      executable=None,
      stdin=None,
      stdout=None,
      stderr=None,
      preexec_fn=None,
      close_fds=False,
      shell=False,
      cwd=None,
      env=None,
      universal_newlines=False,
      startupinfo=None,
      creationflags=0)
```

## 参数

1. ``args``可以是字符串或者序列类型（如：list，元组），用于指定进程的可执行文件及其参数。如果是序列类型，第一个元素通常是可执行文件的路径。我们也可以显式的使用executeable参数来指定可执行文件的路径如`["ls -l"]`，相当于传入cmd。
2. ``bufsize``：指定缓冲。0 无缓冲,1 行缓冲,其他 缓冲区大小,负值 系统缓冲(全缓冲)
3. ``stdin``,`` stdout``, ``stderr**``分别表示程序的标准输入、输出、错误句柄。他们可以是PIPE，文件描述符或文件对象，也可以设置为None，表示从父进程继承。
4. ``preexec_fn``只在Unix平台下有效，用于指定一个可执行对象（callable object），它将在子进程运行之前被调用。
5. ``Close_sfs``：在windows平台下，如果close_fds被设置为True，则新创建的子进程将不会继承父进程的输入、输出、错误管道。我们不能将close_fds设置为True同时重定向子进程的标准输入、输出与错误(stdin, stdout, stderr)。
6. ``shell``设为true，程序将通过shell来执行。
7. ``cwd``用于设置子进程的当前目录
8. ``env``是字典类型，用于指定子进程的环境变量。如果env = None，子进程的环境变量将从父进程中继承。
    Universal_newlines:不同操作系统下，文本的换行符是不一样的。如：windows下用'/r/n'表示换，而Linux下用'/n'。如果将此参数设置为True，Python统一把这些换行符当作'/n'来处理。startupinfo与createionflags只在windows下用效，它们将被传递给底层的CreateProcess()函数，用于设置子进程的一些属性，如：主窗口的外观，进程的优先级等等。
9. ``startupinfo``与``createionflags``只在windows下有效**，它们将被传递给底层的CreateProcess()函数，用于设置子进程的一些属性，如：主窗口的外观，进程的优先级等等。

## 方法

1. ``Popen.poll()``：用于检查子进程是否已经结束。设置并返回returncode属性。
2. ``Popen.wait()``：等待子进程结束。设置并返回returncode属性。
3. `Popen.communicate(input=None)`：与子进程进行交互。向stdin发送数据，或从stdout和stderr中读取数据。可选参数input指定发送到子进程的参数。Communicate()返回一个元组：(stdoutdata, stderrdata)。注意：如果希望通过进程的stdin向其发送数据，在创建Popen对象的时候，参数stdin必须被设置为PIPE。同样，如果希望从stdout和stderr获取数据，必须将stdout和stderr设置为PIPE。
4. `Popen.send_signal(signal)`：向子进程发送信号。
5. `Popen.terminate()`：停止(stop)子进程。在windows平台下，该方法将调用Windows API TerminateProcess（）来结束子进程。
6. `Popen.kill()`：杀死子进程。
7. `Popen.stdin()`：如果在创建Popen对象是，参数stdin被设置为PIPE，Popen.stdin将返回一个文件对象用于策子进程发送指令。否则返回None。
8. `Popen.stdout()`：如果在创建Popen对象是，参数stdout被设置为PIPE，Popen.stdout将返回一个文件对象用于策子进程发送指令。否则返回None。
9. `Popen.stderr()`：如果在创建Popen对象是，参数stdout被设置为PIPE，Popen.stdout将返回一个文件对象用于策子进程发送指令。否则返回None。
10. `Popen.pid()`：获取子进程的进程ID。
11. `Popen.returncode()`：获取进程的返回值。如果进程还没有结束，返回None。
12. `subprocess.call(*popenargs, **kwargs)`：运行命令。该函数将一直等待到子进程运行结束，并返回进程的returncode。文章一开始的例子就演示了call函数。如果子进程不需要进行交互,就可以使用该函数来创建。
13. `subprocess.check_call(*popenargs, **kwargs)`：与subprocess.call(*popenargs, kwargs)功能一样，只是如果子进程返回的returncode不为0的话，将触发CalledProcessError异常。在异常对象中，包括进程的returncode信息。

# 使用例程

```python
# 两者的区别是前者无阻塞,会和主程序并行运行,后者必须等待命令执行完毕
subprocess.Popen('脚本/shell', shell=True)
subprocess.call('脚本/shell', shell=True)

# 有阻塞写法
p = subprocess.Popen('脚本/shell', shell=True)
p.wait()

# 返回结果
p = subprocess.Popen('ls -l', shell=True, stdout=subprocess.PIPE)
p.communicate()   # 返回元组 (stdoutdata, stderrdata),
# communicate()是Popen对象的一个方法，该方法会阻塞父进程，直到子进程完成。
# or
s = subprocess.check_output('ls -l', shell=True)

# 给子进程输入
child = subprocess.Popen(["cat"], stdin=subprocess.PIPE)
child.communicate("vamei")

# pipe
child1 = subprocess.Popen(["ls","-l"], stdout=subprocess.PIPE)
child2 = subprocess.Popen(["wc"], stdin=child1.stdout,stdout=subprocess.PIPE)
out = child2.communicate()
print out
# equal to 
# child1.stdout-->subprocess.PIPE
# child2.stdin<--subprocess.PIPE
# child2.stdout-->subprocess.PIPE
```



