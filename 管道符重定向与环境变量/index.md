# 管道符、重定向与环境变量

##### **3.1 输入输出重定向**

既然已经在上一章学完了几乎所有基础且常用的Linux命令，那么接下来的任务就是把多个Linux命令适当地组合到一起，使其协同工作，以便我们更加高效地处理数据。要做到这一点，就必须搞明白命令的输入重定向和输出重定向的原理。

简而言之，输入重定向是指把文件导入到命令中，而输出重定向则是指把原本要输出到屏幕的数据信息写入到指定文件中。在日常的学习和工作中，相较于输入重定向，我们使用输出重定向的频率更高，所以又将输出重定向分为了标准输出重定向和错误输出重定向两种不同的技术，以及清空写入与追加写入两种模式。听起来就很玄妙？[刘遄](https://www.linuxprobe.com/)老师接下来将慢慢道来。

> 标准输入重定向（STDIN，文件描述符为0）：默认从键盘输入，也可从其他文件或命令中输入。
>
> 标准输出重定向（STDOUT，文件描述符为1）：默认输出到屏幕。
>
> 错误输出重定向（STDERR，文件描述符为2）：默认输出到屏幕。

比如分别查看两个文件的属性信息，先创建出第一个文件，而第二个文件是不存在的。所以虽然针对这两个文件的操作都分别会在屏幕上输出一些信息，但这两个操作的差异其实很大：

```
[root@linuxprobe ~]# touch linuxprobe
[root@linuxprobe ~]# ls -l linuxprobe 
-rw-r--r--. 1 root root 0 Aug 5 05:35 linuxprobe
[root@linuxprobe ~]# ls -l xxxxxx
ls: cannot access xxxxxx: No such file or directory
```

在上述命令中，名为linuxprobe的文件是真实存在的，输出信息是该文件的一些相关权限、所有者、所属组、文件大小及修改时间等信息，这也是该命令的标准输出信息。而名为xxxxxx的第二个文件是不存在的，因此在执行完ls命令之后显示的报错提示信息也是该命令的错误输出信息。那么，要想把原本输出到屏幕上的数据转而写入到文件当中，就要区别对待这两种输出信息。

对于输入重定向来讲，用到的符号及其作用如表3-1所示。

表3-1                     输入重定向中用到的符号及其作用

| 符号                 | 作用                                         |
| -------------------- | -------------------------------------------- |
| 命令 < 文件          | 将文件作为命令的标准输入                     |
| 命令 << 分界符       | 从标准输入中读入，直到遇见分界符才停止       |
| 命令 < 文件1 > 文件2 | 将文件1作为命令的标准输入并将标准输出到文件2 |

对于输出重定向来讲，用到的符号及其作用如表3-2所示。

表3-2                     输出重定向中用到的符号及其作用

| 符号                               | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| 命令 > 文件                        | 将标准输出重定向到一个文件中（清空原有文件的数据）           |
| 命令 2> 文件                       | 将错误输出重定向到一个文件中（清空原有文件的数据）           |
| 命令 >> 文件                       | 将标准输出重定向到一个文件中（追加到原有内容的后面）         |
| 命令 2>> 文件                      | 将错误输出重定向到一个文件中（追加到原有内容的后面）         |
| 命令 >> 文件 2>&1 或 命令 &>> 文件 | 将标准输出与错误输出共同写入到文件中（追加到原有内容的后面） |

对于重定向中的标准输出模式，可以省略文件描述符1不写，而错误输出模式的文件描述符2是必须要写的。先来小试牛刀，通过标准输出重定向将man bash命令原本要输出到屏幕的信息写入到文件readme.txt中，然后显示readme.txt文件中的内容。具体命令如下：

```
[root@linuxprobe ~]# man bash > readme.txt
[root@linuxprobe ~]# cat readme.txt 
BASH(1)                     General Commands Manual                    BASH(1)

NAME
       bash - GNU Bourne-Again SHell

SYNOPSIS
       bash [options] [command_string | file]

COPYRIGHT
       Bash is Copyright (C) 1989-2016 by the Free Software Foundation, Inc.

DESCRIPTION
       Bash  is  an  sh-compatible  command language interpreter that executes
       commands read from the standard input or from a file.  Bash also incor‐
       porates useful features from the Korn and C shells (ksh and csh).

       Bash  is  intended  to  be a conformant implementation of the Shell and
       Utilities portion  of  the  IEEE  POSIX  specification  (IEEE  Standard
       1003.1).  Bash can be configured to be POSIX-conformant by default.

………………省略部分输出信息………………
```

有没有感觉到很方便呢？接下来尝试输出重定向技术中的覆盖写入与追加写入这两种不同模式带来的变化。首先通过覆盖写入模式向readme.txt文件写入多行数据（该文件中已包含上一个实验的man命令信息），但每一次都会覆盖掉上一次的输出内容，所以最终文件中只有最后一次的输出结果：

```
[root@linuxprobe ~]# echo "Welcome to LinuxProbe.Com" > readme.txt
[root@linuxprobe ~]# echo "Welcome to LinuxProbe.Com" > readme.txt
[root@linuxprobe ~]# echo "Welcome to LinuxProbe.Com" > readme.txt
[root@linuxprobe ~]# echo "Welcome to LinuxProbe.Com" > readme.txt
[root@linuxprobe ~]# echo "Welcome to LinuxProbe.Com" > readme.txt
[root@linuxprobe ~]# cat readme.txt
Welcome to LinuxProbe.Com
```

然后再通过追加写入模式向文件再写入一次数据，在执行cat命令之后，可以看到如下所示的文件内容：

```
[root@linuxprobe ~]# echo "Quality linux learning materials" >> readme.txt
[root@linuxprobe ~]# cat readme.txt
Welcome to LinuxProbe.Com
Quality linux learning materials
```

虽然都是输出重定向技术，但是命令的标准输出和错误输出还是有区别的。例如查看当前目录中某个文件的信息，这里以linuxprobe文件为例，这个文件是真实存在的，因此使用标准输出即可将原本要输出到屏幕的信息写入到文件中，而错误的输出重定向则依然把信息输出到了屏幕上。

```
[root@linuxprobe ~]# ls -l linuxprobe > /root/stderr.txt 
[root@linuxprobe ~]# ls -l linuxprobe 2> /root/stderr.txt 
-rw-r--r--. 1 root root 0 Mar  1 13:30 linuxprobe
```

如果想把命令的报错信息写入到文件，该怎么操作呢？当用户在执行一个自动化的Shell[脚本](https://www.linuxcool.com/)时，这个操作会特别有用，而且特别实用，因为它可以把整个脚本执行过程中的报错信息都记录到文件中，便于安装后的排错工作。

接下来以一个不存在的文件进行实验演示：

```
[root@linuxprobe ~]# ls -l xxxxxx > /root/stderr.txt
cannot access xxxxxx: No such file or directory
[root@linuxprobe ~]# ls -l xxxxxx 2> /root/stderr.txt
[root@linuxprobe ~]# cat /root/stderr.txt 
ls: cannot access xxxxxx: No such file or directory
```

还有一种常见情况，就是我们想不区分标准还是错误输出信息，只要命令有输出信息则全部追加写入到文件中，这种就要使用到&>>操作符了，全部写入到文件中：

```
[root@linuxprobe ~]# ls -l linuxprobe &>> readme.txt
[root@linuxprobe ~]# ls -l xxxxxx &>> readme.txt
-rw-r--r--. 1 root root 0 Mar  1 13:30 linuxprobe
cannot access xxxxxx: No such file or directory
```

输入重定向相对来说有些冷门，在工作中遇到的概率会小一点。输入重定向的作用是把文件直接导入到命令中。接下来使用输入重定向把readme.txt文件导入给wc -l命令，统计一下文件中的内容行数：

```
[root@linuxprobe ~]# wc -l < readme.txt
2
```

大家肯定发现了这次的输出结果跟第2章节讲的时候有所不同，没有了文件名称。

```
[root@linuxprobe ~]# wc -l /etc/passwd
38 /etc/passwd
```

这是因为此前使用“wc -l /etc/passwd”是一种非常标准的“命令+参数+对象”的执行格式，而这次的“wc -l < readme.txt”则是将readme.txt文件内的内容通过操作符导入到的命令中，没有被当作命令对象进行执行的过程，因此wc命令只能读到信息流的数据，没有文件名称的信息。这个小差异，同学们可以慢慢琢磨下。

##### **3.2 管道命令符**

细心的读者肯定还记得在2.6节学习tr命令时曾经见到过一个名为管道符的东西。同时按下键盘上的“Shift”和“\”反斜杠键即可打出管道符，其执行格式为“命令A | 命令B”。管道命令符的作用也可以用一句话来概括“**把前一个命令原本要输出到屏幕的信息当作是后一个命令的标准输入**”。在2.6节讲解grep文本搜索命令时，我们通过匹配关键词/sbin/nologin找出了所有被限制登录系统的用户。在学完本节内容后，完全可以把下面这两条命令合并为一条：

> 找出被限制登录用户的命令是：grep /sbin/nologin /etc/passwd
>
> 统计文本行数的命令则是：wc -l

现在要做的就是把grep搜索命令的输出值传递给wc统计命令，即把原本要输出到屏幕的用户信息列表再交给wc命令作进一步的加工，因此只需要把管道符放到两条命令之间即可，具体如下。这简直是太方便了！

```
[root@linuxprobe ~]# grep /sbin/nologin /etc/passwd | wc -l
40
```

这个管道符就像一个法宝，可以将它套用到其他不同的命令上，比如用翻页的形式查看/etc目录中的文件列表及属性信息（这些内容默认会一股脑儿地显示到屏幕上，根本看不清楚）：

```
[root@linuxprobe ~]# ls -l /etc/ | more
total 1344
-rw-r--r--. 1 root root 16 Jul 21 05:08 adjtime
-rw-r--r--. 1 root root 1518 Sep 10 2018 aliases
drwxr-xr-x. 3 root root 65 Jul 21 05:06 alsa
drwxr-xr-x. 2 root root 4096 Jul 21 05:08 alternatives
-rw-r--r--. 1 root root 541 Oct 2 2018 anacrontab
-rw-r--r--. 1 root root 55 Feb 1 2019 asound.conf
-rw-r--r--. 1 root root 1 Aug 12 2018 at.deny
drwxr-x---. 4 root root 100 Jul 21 05:16 audit
drwxr-xr-x. 3 root root 228 Jul 21 05:08 authselect
drwxr-xr-x. 4 root root 71 Jul 21 05:06 avahi
drwxr-xr-x. 2 root root 204 Jul 21 05:06 bash_completion.d
-rw-r--r--. 1 root root 3001 Sep 10 2018 bashrc
--More--
```

在修改用户密码时，通常都需要输入两次密码以进行确认，这在编写自动化脚本时将成为一个非常致命的缺陷。通过把管道符和passwd命令的--stdin参数相结合，可以用一条命令来完成密码重置操作：

```
[root@linuxprobe ~]# echo "linuxprobe" | passwd --stdin root
Changing password for user root.
passwd: all authentication tokens updated successfully.
```

还有咱们上个章节学习ps命令的时候，输入ps aux参数后屏幕信息呼呼闪过，根本找不到有用的信息，现在也可以将ps、grep、管道符三者结合到一起使用了。搜索与bash有关的进程信息：

```
[root@linuxprobe ~]# ps aux | grep bash
root 1070 0.0 0.1 25384 2324 ? S Sep21 0:00 /bin/bash /usr/sbin/ksmtuned
root 3899 0.0 0.2 26540 5136 pts/0 Ss 00:27 0:00 bash
root 4002 0.0 0.0 12112 1056 pts/0 S+ 00:28 0:00 grep --color=auto bash
```

![第3章 管道符、重定向与环境变量第3章 管道符、重定向与环境变量](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E4%BB%BB%E6%84%8F%E9%97%A8.gif)

 

当然，大家千万不要误以为管道命令符只能在一个命令组合中使用一次，我们完全可以这样使用：“命令A | 命令B | 命令C”。为了帮助读者进一步理解管道符的作用，刘遄老师在讲课时经常会把管道符描述成“任意门”。想必大家小时候都看过“哆啦A梦”动画片吧。哆啦A梦（也就是常称的机器猫）经常为了取悦大雄而从口袋中掏出一件件宝贝，其中好多次就用到了任意门这个道具。其实，管道符就好像是用于实现数据穿越的任意门，能够帮助提高工作效率，完成之前不敢想象的复杂工作。

 

### **Tips**

曾经有位东北的同学讲了个特别贴切的例子，把管道符比喻成流水线作业，跟吃了顿烧烤是一个道理，第一个人负责切肉，第二个人负责串肉，第三个人负责烧烤，最后的处理结果交付给用户~

如果读者是一名Linux新手，可能会觉得上面的命令组合已经十分复杂了，但是有过运维经验的读者又会感觉如隔靴挠痒般不过瘾，他们希望能将这样方便的命令写得更高级一些，功能更强大一些,为了大家对我们这本书的捧场，当然要义不容辞地把技术拱手奉上。如果需要将管道符处理后的结果即输出到屏幕，又同时写入到文件中，则可以与tee命令结合使用。

显示系统中所有与bash相关的进程信息，结果同时输出到屏幕和文件：

![第3章 管道符、重定向与环境变量第3章 管道符、重定向与环境变量](https://www.linuxprobe.com/wp-content/uploads/2020/05/netstat.png)

```
[root@linuxprobe ~]# ps aux | grep bash | tee result.txt
root 1070 0.0 0.1 25384 2324 ? S Sep21 0:00 /bin/bash /usr/sbin/ksmtuned
root 3899 0.0 0.2 26540 5136 pts/0 Ss 00:27 0:00 bash
root 4320 0.0 0.0 12112 1112 pts/0 S+ 00:51 0:00 grep --color=auto bash
[root@linuxprobe ~]# cat result.txt
root 1070 0.0 0.1 25384 2324 ? S Sep21 0:00 /bin/bash /usr/sbin/ksmtuned
root 3899 0.0 0.2 26540 5136 pts/0 Ss 00:27 0:00 bash
root 4320 0.0 0.0 12112 1112 pts/0 S+ 00:51 0:00 grep --color=auto bash
```

##### **3.3 命令行的通配符**

大家可能都遇到过提笔忘字的尴尬，作为Linux运维人员，有时候也会遇到明明一个文件的名称就在嘴边但就是想不起来的情况。如果就记得一个文件的开头几个字母，想遍历查找出所有以这个关键词开头的文件，该怎么操作呢？又假设想要批量查看所有硬盘文件的相关权限属性，一种方式是这样的：

```
[root@linuxprobe ~]# ls -l /dev/sda
brw-rw----. 1 root disk 8, 0 May 4 15:55 /dev/sda
[root@linuxprobe ~]# ls -l /dev/sda1
brw-rw----. 1 root disk 8, 1 May 4 15:55 /dev/sda1
[root@linuxprobe ~]# ls -l /dev/sda2
brw-rw----. 1 root disk 8, 2 May 4 15:55 /dev/sda2
[root@linuxprobe ~]# ls -l /dev/sda3
ls: cannot access '/dev/sda3': No such file or directory
```

幸亏我的硬盘文件和分区只有3个，要是有几百个，估计需要花费一天的时间来忙这个事情了，所以这种方式的效率确实很低。

虽然在第6章才会讲解Linux系统的存储结构和FHS，但现在应该能看出一些简单规律了。比如，这些硬盘设备文件都是以sda开头并且存放到了/dev目录中，这样一来，即使不知道硬盘的分区编号和具体分区的个数，也可以使用通配符来搞定。

顾名思义，通配符就是通用的匹配信息的符号，比如星号（*）代表匹配零个或多个字符，问号（?）代表匹配单个字符，中括号内加上数字[0-9]代表匹配0～9之间的单个数字的字符，而中括号内加上字母[abc]则是代表匹配a、b、c三个字符中的任意一个字符。Linux系统中的通配符及含义如表3-3所示：

表3-3                    Linux系统中的通配符及含义

| 通配符    | 含义           |
| --------- | -------------- |
| *         | 任意字符       |
| ?         | 单个任意字符   |
| [a-z]     | 单个小写字母   |
| [A-Z]     | 单个大写字母   |
| [a-Z]     | 单个字母       |
| [0-9]     | 单个数字       |
| [:alpha:] | 任意字母       |
| [:upper:] | 任意大写字母   |
| [:lower:] | 任意小写字母   |
| [:digit:] | 所有数字       |
| [:alnum:] | 任意字母加数字 |
| [:punct:] | 标点符号       |

俗话讲“百闻不如一见，看书不如实验”，下面我们就来匹配所有在/dev目录中且以sda开头的文件：

```
[root@linuxprobe ~]# ls -l /dev/sda*
brw-rw----. 1 root disk 8, 0 May 4 15:55 /dev/sda
brw-rw----. 1 root disk 8, 1 May 4 15:55 /dev/sda1
brw-rw----. 1 root disk 8, 2 May 4 15:55 /dev/sda2
```

如果只想查看文件名为sda开头，但是后面还紧跟其他某一个字符的文件的相关信息，这时就需要用到问号来进行通配了：

```
[root@linuxprobe ~]# ls -l /dev/sda?
brw-rw----. 1 root disk 8, 1 May 4 15:55 /dev/sda1
brw-rw----. 1 root disk 8, 2 May 4 15:55 /dev/sda2
```

除了使用[0-9]来匹配0~9之间的单个数字，也可以用[135]这样的方式仅匹配这三个指定数字中的一个，若没有匹配到数字1或2或3则不会显示出来：

```
[root@linuxprobe ~]# ls -l /dev/sda[0-9]
brw-rw----. 1 root disk 8, 1 May 4 15:55 /dev/sda1
brw-rw----. 1 root disk 8, 2 May 4 15:55 /dev/sda2
[root@linuxprobe ~]# ls -l /dev/sda[135]
brw-rw----. 1 root disk 8, 1 May 4 15:55 /dev/sda1
```

通配符不一定非要放到最后面，也可以搜索下/etc/目录中所有以.conf结尾的配置文件有哪些：

```
[root@linuxprobe ~]# ls -l /etc/*.conf
-rw-r--r--. 1 root root 55 Feb 1 2019 /etc/asound.conf
-rw-r--r--. 1 root root 25696 Dec 12 2018 /etc/brltty.conf
-rw-r--r--. 1 root root 1083 Apr 4 2018 /etc/chrony.conf
-rw-r--r--. 1 root root 1174 Aug 12 2018 /etc/dleyna-server-service.conf
-rw-r--r--. 1 root dnsmasq 26843 Aug 12 2018 /etc/dnsmasq.conf
-rw-r--r--. 1 root root 117 Jan 16 2019 /etc/dracut.conf
-rw-r--r--. 1 root root 20 Aug 12 2018 /etc/fprintd.conf
-rw-r--r--. 1 root root 38 Nov 16 2018 /etc/fuse.conf
………………省略部分输出信息………………
```

通配符不仅被用于搜索文件或代替被通配的字符，还可以与创建文件的命令相结合，一口气创建出好多个文件。唯一区别是要用到大括号，并且字段之间用逗号间隔：

```
[root@linuxprobe ~]# touch {AA,BB,CC}.conf
[root@linuxprobe ~]# ls -l *.conf
-rw-r--r--. 1 root root 0 Sep 22 01:54 AA.conf
-rw-r--r--. 1 root root 0 Sep 22 01:54 BB.conf
-rw-r--r--. 1 root root 0 Sep 22 01:54 CC.conf
```

再或者输出一些指定的信息，玩法特别多，接下来大家就自己开发吧~：

```
[root@linuxprobe ~]# echo file{1,2,3,4,5}
file1 file2 file3 file4 file5
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **3.4 常用的转义字符**

为了能够更好地理解用户的表达，Shell解释器还提供了特别丰富的转义字符来处理输入的特殊数据。刘遄老师以近十年的工作和培训为基础，愣是用了两周时间从数十个转义字符中提炼出了4个最常用的转义字符！这件事情也让我深刻反省了很长时间。原本认为图书写的越厚，作者越是大牛，现在发现这种观念完全是错误的，希望读者在读完本书后能体会到刘遄老师的用心付出。

4个最常用的转义字符如下所示。

> 反斜杠（\）：使反斜杠后面的一个变量变为单纯的字符。
>
> 单引号（''）：转义其中所有的变量为单纯的字符串。
>
> 双引号（""）：保留其中的变量属性，不进行转义处理。
>
> 反引号（``）：把其中的命令执行后返回结果。

先定义一个名为PRICE的变量并赋值为5，然后输出以双引号括起来的字符串与变量信息：

```
[root@linuxprobe ~]# PRICE=5
[root@linuxprobe ~]# echo "Price is $PRICE"
Price is 5
```

接下来，我们希望能够输出“Price is $5”，即价格是5美元的字符串内容，但碰巧美元符号与变量提取符号合并后的$$作用是显示当前程序的进程ID号码，于是命令执行后输出的内容并不是我们所预期的：

```
[root@linuxprobe ~]# echo "Price is $$PRICE" 
Price is 3767PRICE
```

要想让第一个“$”乖乖地作为美元符号，那么就需要使用反斜杠（\）来进行转义，将这个命令提取符转义成单纯的文本，去除其特殊功能：

```
[root@linuxprobe ~]# echo "Price is \$$PRICE"
Price is $5
```

而如果只需要某个命令的输出值时，可以像`命令`这样，将命令用反引号括起来，达到预期的效果。例如，将反引号与uname -a命令结合，然后使用echo命令来查看本机的Linux版本和内核信息：

```
[root@linuxprobe ~]# echo `uname -a`
Linux linuxprobe.com 4.18.0-80.el8.x86_64 #1 SMP Wed Mar 13 12:02:46 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

反斜杠和反引号的功能比较有特点，同学们一般不会犯错，但对于什么时候使用双引号却容易混淆，因为好像大多数情况下加不加效果都一样：

```
[root@linuxprobe ~]# echo AA BB CC
AA BB CC
[root@linuxprobe ~]# echo "AA BB CC"
AA BB CC
```

区别在于用户无法得知第一种执行方式中到底有几个参数，是的，不能确定！因为有可能把“AA BB CC”当作了一个参数整体直接输出到了屏幕，也有可能是分别将AA、BB和CC输出到了屏幕，就算摸清了echo命令处理参数的机制，其他命令依然是这种情况。因此给大家总结一个简单小技巧，虽然可能不够严谨，但绝对简单，就是参数中如果出现了空格，那么就加双引号，如果参数中没有空格，那就不用加~

##### **3.5 重要的环境变量**

变量是计算机系统用于保存可变值的数据类型。在Linux系统中，变量名称一般都是大写的，命令则都是小写的，这是一种约定俗成的规范。Linux系统中的环境变量是用来定义系统运行环境的一些参数，比如每个用户不同的家目录、邮件存放位置等，可以直接通过变量名称来提取到对应的变量值。

细心的读者应该发现了，本节和上一节的标题名都分别加了形容词—重要的、常见的。原因其实不言而喻—要想让Linux系统能够正常运行并且为用户提供服务，需要数百个环境变量来协同工作，我们没有必要逐一查看、学习每一个变量，而是应该在有限的篇幅中精讲最重要的内容。

为了更好地帮助大家理解变量的作用，给大家举个例子。前文中曾经讲到，在Linux系统中一切都是文件，Linux命令也不例外。那么，在用户执行了一条命令之后，Linux系统中到底发生了什么事情呢？简单来说，命令在Linux中的执行分为四个步骤。

**第1步**：判断用户是否以绝对路径或相对路径的方式输入命令（如/bin/ls），如果是的话则直接执行。

**第2步**：Linux系统检查用户输入的命令是否为“别名命令”，即用一个自定义的命令名称来替换原本的命令名称。

之前在使用rm命令删除文件时，Linux系统都会要求用户再确认是否执行删除操作，其实这就是Linux系统为了防止用户误删除文件而特意设置的rm别名命令。

```
[root@linuxprobe ~]# ls
anaconda-ks.cfg  Documents  initial-setup-ks.cfg  Pictures  Templates
Desktop          Downloads  Music                 Public    Videos
[root@linuxprobe ~]# rm anaconda-ks.cfg 
rm: remove regular file 'anaconda-ks.cfg'? y
```

也可以用alias命令来创建一个属于自己的命令别名，语法格式为“alias 别名=命令”，若要取消一个命令别名，则是用unalias命令，语法格式为“unalias 别名”。

将当前rm命令所被设置的别名取消掉，再删除文件试试：

```
[root@linuxprobe ~]# unalias rm
[root@linuxprobe ~]# rm initial-setup-ks.cfg 
[root@linuxprobe ~]#
```

**第3步**：Bash解释器判断用户输入的是内部命令还是外部命令。内部命令是解释器内部的指令，会被直接执行；而用户在绝大部分时间输入的是外部命令，这些命令交由步骤4继续处理。可以使用“type 命令名称”来判断用户输入的命令是内部命令还是外部命令：

```
[root@linuxprobe ~]# type echo
echo is a shell builtin
[root@linuxprobe ~]# type uptime
uptime is /usr/bin/uptime
```

**第4步**：系统在多个路径中查找用户输入的命令文件，而定义这些路径的变量叫作PATH，可以简单地把它理解成是“解释器的小助手”，作用是告诉Bash解释器待执行的命令可能存放的位置，然后Bash解释器就会乖乖地在这些位置中逐个查找。PATH是由多个路径值组成的变量，每个路径值之间用冒号间隔，对这些路径的增加和删除操作将影响到Bash解释器对Linux命令的查找。

```
[root@linuxprobe ~]# echo $PATH
/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/root/bin
[root@linuxprobe ~]# PATH=$PATH:/root/bin
[root@linuxprobe ~]# echo $PATH
/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/root/bin:/root/bin
```

这里有比较经典的问题：“为什么不能将当前目录（.）添加到PATH中呢? ” 原因是，尽管可以将当前目录（.）添加到PATH变量中，从而在某些情况下可以让用户免去输入命令所在路径的麻烦。但是，如果黑客在比较常用的公共目录/tmp中存放了一个与ls或cd命令同名的木马文件，而用户又恰巧在公共目录中执行了这些命令，那么就极有可能中招了。

所以，作为一名态度谨慎、有经验的运维人员，在接手了一台Linux系统后一定会在执行命令前先检查PATH变量中是否有可疑的目录，另外读者从前面的PATH变量示例中是否也感觉到环境变量特别有用呢。我们可以使用env命令来查看到Linux系统中所有的环境变量，而刘遄老师为您精挑细选出了最重要的10个环境变量，如表3-4所示。

表3-4                    Linux系统中最重要的10个环境变量

| 变量名称     | 作用                             |
| ------------ | -------------------------------- |
| HOME         | 用户的主目录（即家目录）         |
| SHELL        | 用户在使用的Shell解释器名称      |
| HISTSIZE     | 输出的历史命令记录条数           |
| HISTFILESIZE | 保存的历史命令记录条数           |
| MAIL         | 邮件保存路径                     |
| LANG         | 系统语言、语系名称               |
| RANDOM       | 生成一个随机数字                 |
| PS1          | Bash解释器的提示符               |
| PATH         | 定义解释器搜索用户执行命令的路径 |
| EDITOR       | 用户默认的文本编辑器             |

Linux作为一个多用户多任务的操作系统，能够为每个用户提供独立的、合适的工作运行环境，因此，一个相同的变量会因为用户身份的不同而具有不同的值。例如，使用下述命令来查看HOME变量在不同用户身份下都有哪些值（su是用于切换用户身份的命令，将在第5章跟大家见面）：

```
[root@linuxprobe ~]# echo $HOME
/root
[root@linuxprobe ~]# su - linuxprobe
[linuxprobe@linuxprobe ~]$ echo $HOME
/home/linuxprobe
```

其实变量是由固定的变量名与用户或系统设置的变量值两部分组成的，完全可以自行创建变量，来满足工作需求。例如设置一个名称为WORKDIR的变量，方便用户更轻松地进入一个层次较深的目录：

```
[root@linuxprobe ~]# mkdir /home/workdir
[root@linuxprobe ~]# WORKDIR=/home/workdir
[root@linuxprobe ~]# cd $WORKDIR 
[root@linuxprobe workdir]# pwd
/home/workdir
```

但是，这样的变量不具有全局性，作用范围也有限，默认情况下不能被其他用户使用：

```
[root@linuxprobe workdir]# su linuxprobe
[linuxprobe@linuxprobe ~]$ cd $WORKDIR
[linuxprobe@linuxprobe ~]$ echo $WORKDIR
[linuxprobe@linuxprobe ~]$ exit
```

如果工作需要，可以使用export命令将其提升为全局变量，这样其他用户也就可以使用它了：

```
[root@linuxprobe ~]# export WORKDIR
[root@linuxprobe ~]# su linuxprobe
[linuxprobe@linuxprobe ~]$ cd $WORKDIR
[linuxprobe@linuxprobe workdir]$ pwd
/home/workdir
```

后续要是不用这个变量了，那么就顺手执行下unset命令取消掉它吧：

```
[root@linuxprobe ~]# unset WORKDIR
[root@linuxprobe ~]#
```

### **Tips**

直接在终端设置的变量能够立即生效，但重启服务器后就会消失掉，因此我们需要将变量和变量值写入到.bashrc或者.bash_profile文件中以确保永久能使用它们。什么？不知道该怎么编辑文件吗？快来看第四章节吧~~

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．把ls命令的正常输出信息追加写入到error.txt文件中的命令是什么？

**答：**ls >> error.txt，注意区分>和>>的不同。

2．请简单概述管道符的作用。

**答：**把左面（前面）命令的输出值作为右面（后面）命令的输入值以便进一步处理信息。

3．Bash解释器的通配符中，星号（*）代表几个字符？

**答：**零个或多个均可。

4．PATH变量的作用是什么？

**答：**设定解释器搜索所执行命令的路径，找到其所在位置。

5．一般情况下，对参数添加双引号有什么好处？

**答：**通常用于界定参数的个数，避免程序或命令执行时产生歧义，因此参数中若有空格则建议添加上双引号。

6．使用什么命令可以把名为LINUX的一般变量转换成全局变量？

**答：**export LINUX。

