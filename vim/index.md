# vim

##### **4.1 Vim文本编辑器**

Vim的发布最早可以追溯到1991年，英文全称为：“Vi IMproved”，也就是对vi编辑器的提升版本，其中最大的改进当属对代码的着色功能，亦有些编程场景能够自动修正错误代码。

每当在讲课时遇到需要让学生记住的知识点时，为了能让他们打起精神来，我都会突然提高嗓门，因此有句话他们记得尤其深刻：“**在[Linux系统](https://www.linuxprobe.com/)中一切都是文件，而配置一个服务就是在修改其配置文件的参数**”。而且在日常工作中大家也肯定免不了要编写文档，这些工作都是通过文本编辑器来完成的。[刘遄](https://www.linuxprobe.com/)老师写作本书的目的是让读者切实掌握Linux系统的运维方法，而不是仅仅停留在“会用某个操作系统”的层面上，所以我们这里选择使用Vim文本编辑器，它默认会安装在当前所有的Linux操作系统上，是一款超棒的文本编辑器。

Vim之所以能得到广大厂商与用户的认可，原因在于Vim编辑器中设置了三种模式—命令模式、末行模式和编辑模式，每种模式分别又支持多种不同的命令快捷键，这大大提高了工作效率，而且用户在习惯之后也会觉得相当顺手。要想高效率地操作文本，就必须先搞清这三种模式的操作区别以及模式之间的切换方法（见图4-1）。

> 命令模式：控制光标移动，可对文本进行复制、粘贴、删除和查找等工作。
>
> 输入模式：正常的文本录入。
>
> 末行模式：保存或退出文档，以及设置编辑环境。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2015/03/vim%E4%B8%8D%E5%90%8C%E6%A8%A1%E5%BC%8F%E9%97%B4%E7%9A%84%E5%88%87%E6%8D%A2.png)

图4-1 Vim编辑器模式的切换方法

在每次运行Vim编辑器时，默认进入命令模式，此时需要先切换到输入模式后再进行文档编写工作，而每次在编写完文档后需要先返回命令模式，然后再进入末行模式，执行文档的保存或退出操作。在Vim中，无法直接从输入模式切换到末行模式。Vim编辑器中内置的命令有成百上千种用法，为了能够帮助读者更快地掌握Vim编辑器，表4-1总结了在命令模式中最常用的一些命令。

表4-1                          命令模式中最常用的一些命令

| 命令 | 作用                                               |
| ---- | -------------------------------------------------- |
| dd   | 删除（剪切）光标所在整行                           |
| 5dd  | 删除（剪切）从光标处开始的5行                      |
| yy   | 复制光标所在整行                                   |
| 5yy  | 复制从光标处开始的5行                              |
| n    | 显示搜索命令定位到的下一个字符串                   |
| N    | 显示搜索命令定位到的上一个字符串                   |
| u    | 撤销上一步的操作                                   |
| p    | 将之前删除（dd）或复制（yy）过的数据粘贴到光标后面 |

末行模式主要用于保存或退出文件，以及设置Vim编辑器的工作环境，还可以让用户执行外部的Linux命令或跳转到所编写文档的特定行数。要想切换到末行模式，在命令模式中输入一个冒号就可以了。末行模式中常用的命令如表4-2所示。

表4-2                          末行模式中最常用的一些命令

| 命令          | 作用                                 |
| ------------- | ------------------------------------ |
| :w            | 保存                                 |
| :q            | 退出                                 |
| :q!           | 强制退出（放弃对文档的修改内容）     |
| :wq!          | 强制保存退出                         |
| :set nu       | 显示行号                             |
| :set nonu     | 不显示行号                           |
| :命令         | 执行该命令                           |
| :整数         | 跳转到该行                           |
| :s/one/two    | 将当前光标所在行的第一个one替换成two |
| :s/one/two/g  | 将当前光标所在行的所有one替换成two   |
| :%s/one/two/g | 将全文中的所有one替换成two           |
| ?字符串       | 在文本中从下至上搜索该字符串         |
| /字符串       | 在文本中从上至下搜索该字符串         |

编辑器平日里一定要多用，靠的就是个熟练程度，一旦把Vim的各种命令练熟，再编辑配置文件的时候能比同事快不止一倍。我记得在2011年的时候见过一个叫做Aleksandr Levchuk的极客大咖，就为了准求极致的效率感，而发起了一个叫“VIM Clutch”的实验项目，他买了一对类似于汽车油门和刹车的离合器，改装后再用USB与电脑相连接，左脚刹车是进入编辑模式（i），右脚油门是保存文件（wq!），这种对Linux和Vim的热爱真是强大。

###### **4.1.1 编写简单文档**

目前为止，大家已经具备了在Linux系统中编写文档的理论基础了，接下来我们一起动手编写一个简单的脚本文档。刘遄老师会尽力把所有操作步骤和按键过程都标注出来，如果忘记了某些快捷键命令的作用，可以再返回前文进行复习。

编写脚本文档的第一步就是给文档取个名字，这里将其命名为practice.txt。如果存在该文档，则是打开它。如果不存在，则是创建一个临时的输入文件，如图4-2所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E5%B0%9D%E8%AF%95%E7%BC%96%E5%86%99%E6%96%87%E6%9C%AC%E6%96%87%E6%A1%A3.png)

图4-2 尝试编写文本文档

打开practice.txt文档后，默认进入的是Vim编辑器的命令模式。此时只能执行该模式下的命令，而不能随意输入文本内容，我们需要切换到输入模式才可以编写文档。

在图4-1中提到，可以分别使用a、i、o三个键从命令模式切换到输入模式。其中，a键与i键分别是在光标后面一位和光标当前位置切换到输入模式，而o键则是在光标的下面再创建一个空行，此时可敲击a键进入到编辑器的输入模式，如图4-3所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E5%88%87%E6%8D%A2%E8%87%B3%E7%BC%96%E8%BE%91%E5%99%A8%E7%9A%84%E8%BE%93%E5%85%A5%E6%A8%A1%E5%BC%8F-1.png)

图4-3 切换至编辑器的输入模式

进入输入模式后，可以随意输入文本内容，Vim编辑器不会把您输入的文本内容当作命令而执行，如图4-4所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E5%9C%A8%E7%BC%96%E8%BE%91%E5%99%A8%E4%B8%AD%E8%BE%93%E5%85%A5%E6%96%87%E6%9C%AC%E5%86%85%E5%AE%B9.png)

图4-4 在编辑器中输入文本内容

在编写完之后，想要保存并退出，必须先敲击键盘Esc键从输入模式返回命令模式，如图4-5所示。然后再输入:wq!切换到末行模式才能完成保存退出操作，如图4-6所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E5%88%87%E6%8D%A2%E8%87%B3%E7%BC%96%E8%BE%91%E5%99%A8%E7%9A%84%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F.png)

图4-5 切换至编辑器的命令模式

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E5%88%87%E6%8D%A2%E8%87%B3%E7%BC%96%E8%BE%91%E5%99%A8%E7%9A%84%E6%9C%AB%E8%A1%8C%E6%A8%A1%E5%BC%8F.png)

图4-6 切换至编辑器的末行模式

### **Tips**

同学们仔细观察图4-4、4-5、4-6中左下角的提示信息，不同模式下有不同的提示字样。

当在末行模式中输入:wq!命令时，就意味着强制保存并退出文档。然后便可以用cat命令查看保存后的文档内容了，如图4-7所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E6%9F%A5%E7%9C%8B%E6%96%87%E6%A1%A3%E7%9A%84%E5%86%85%E5%AE%B9.png)

图4-7 查看文档的内容

是不是很简单？！继续编辑这个文档。因为要在原有文本内容的下面追加内容，所以在命令模式中敲击o键进入输入模式更会高效，操作如图4-8、图4-9与图4-10所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E5%86%8D%E6%AC%A1%E9%80%9A%E8%BF%87Vim%E7%BC%96%E8%BE%91%E5%99%A8%E7%BC%96%E5%86%99%E6%96%87%E6%A1%A3.png)

图4-8 再次通过Vim编辑器编写文档

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E8%BF%9B%E5%85%A5Vim%E7%BC%96%E8%BE%91%E5%99%A8%E7%9A%84%E8%BE%93%E5%85%A5%E6%A8%A1%E5%BC%8F.png)

图4-9 进入Vim编辑器的输入模式

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E8%BF%BD%E5%8A%A0%E5%86%99%E5%85%A5%E4%B8%80%E8%A1%8C%E6%96%87%E6%9C%AC%E5%86%85%E5%AE%B9.png)

图4-10 追加写入一行文本内容

因为此时已经修改了文本内容，所以Vim编辑器在尝试直接退出文档而不保存的时候就会拒绝我们的操作了。此时只能强制退出才能结束本次输入操作，如图4-11、图4-12和图4-13所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E9%80%80%E5%87%BA%E6%96%87%E6%9C%AC%E7%BC%96%E8%BE%91%E5%99%A8.png)

图4-11 退出文本编辑器

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E5%9B%A0%E6%96%87%E4%BB%B6%E5%B7%B2%E8%A2%AB%E4%BF%AE%E6%94%B9%E8%80%8C%E6%8B%92%E7%BB%9D%E9%80%80%E5%87%BA%E6%93%8D%E4%BD%9C.png)

图4-12 因文件已被修改而拒绝退出操作

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E5%BC%BA%E5%88%B6%E9%80%80%E5%87%BA%E6%96%87%E6%9C%AC%E7%BC%96%E8%BE%91%E5%99%A8.png)

图4-13  强制退出文本编辑器

现在大家也算是具有了一些Vim编辑器的实战经验了，应该也感觉没有想象中那么难吧。现在查看文本的内容，果然发现追加输入的内容并没有被保存下来，如图4-14所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E6%9F%A5%E7%9C%8B%E6%9C%80%E7%BB%88%E7%BC%96%E5%86%99%E6%88%90%E7%9A%84%E6%96%87%E6%9C%AC%E5%86%85%E5%AE%B9.png)

图4-14  查看最终编写成的文本内容

大家在学完了理论知识之后又自己动手编写了一个文本，现在是否感觉成就满满呢？接下来将会由浅入深为读者安排三个小任务。为了彻底掌握Vim编辑器的使用，大家一定要逐个完成不许偷懒，如果在完成这三个任务期间忘记了相关命令，可返回前文进一步复习掌握。

### **Tips**

下面实验做不成功很正常，请把重心放到Vim编辑器上面，配置文件能修改成功就很棒啦~

###### **4.1.2 配置主机名称**

为了便于在局域网中查找某台特定的主机，或者对主机进行区分，除了要有IP地址外，还要为主机配置一个主机名，主机之间可以通过这个类似于域名的名称来相互访问。在Linux系统中，主机名大多保存在/etc/hostname文件中，接下来将配置文件的内容修改为“linuxprobe.com”，步骤如下。

> **第1步**：使用Vim编辑器修改“/etc/hostname”主机名称文件。
>
> **第2步**：把原始主机名称删除后追加“linuxprobe.com”。注意，使用Vim编辑器修改主机名称文件后，要在末行模式下执行:wq!命令才能保存并退出文档。
>
> **第3步**：保存并退出文档，然后使用hostname命令检查是否修改成功。

```
[root@linuxprobe ~]# vim /etc/hostname
linuxprobe.com
```

hostname命令用于查看当前的主机名称，但有时主机名称的改变不会立即同步到系统中，所以如果发现修改完成后还显示原来的主机名称，可重启虚拟机后再行查看：

```
[root@linuxprobe ~]# hostname
linuxprobe.com
```

###### **4.1.3 配置网卡信息**

网卡IP地址配置的是否正确是两台服务器是否可以相互通信的前提。在Linux系统中，一切都是文件，因此配置网络服务的工作其实就是在编辑网卡配置文件，这个小任务不仅可以帮助您练习使用Vim编辑器，而且也为后面学习Linux中的各种服务配置打下了坚实的基础。当认真学习完本书后，一定会特别有成就感，因为本书前面的基础部分非常扎实，而后面内容则具有几乎一致的网卡IP地址和运行环境，从而确保您全身心地投入到各类服务程序的学习上，而不用操心系统环境的问题。

如果您具备一定的运维经验或者熟悉早期的Linux系统，则在学习本书时会遇到一些不容易接受的差异变化。在RHEL 5、RHEL 6中，网卡配置文件的前缀为eth，第1块网卡为eth0，第2块网卡为eth1；以此类推。而在RHEL 7中，网卡配置文件的前缀则以ifcfg开始，加上网卡名称共同组成了网卡配置文件的名字，例如ifcfg-eno16777736；而在RHEL 8中，网卡配置文件的前缀依然为ifcfg，区别是名称改成了类似于ens160的样子，不过好在除了文件名变化外网卡参数没有其他大的区别。

现在有一个名称为ifcfg-ens160的网卡设备，将其配置为开机自启动，并且IP地址、子网、网关等信息由人工指定，其步骤应该如下所示。

**第1步**：首先切换到/etc/sysconfig/network-scripts目录中（存放着网卡的配置文件）。

**第2步**：使用Vim编辑器修改网卡文件ifcfg-ens160，逐项写入下面的配置参数并保存退出。由于每台设备的硬件及架构是不一样的，因此请读者使用ifconfig命令自行确认各自网卡的默认名称。

> 设备类型：TYPE=Ethernet
>
> 地址分配模式：BOOTPROTO=static
>
> 网卡名称：NAME=ens160
>
> 是否启动：ONBOOT=yes
>
> IP地址：IPADDR=192.168.10.10
>
> 子网掩码：NETMASK=255.255.255.0
>
> 网关地址：GATEWAY=192.168.10.1
>
> DNS地址：DNS1=192.168.10.1

**第3步**：重启网络服务并测试网络是否联通。

好的，正式开干！

进入到网卡配置文件所在的目录，然后编辑网卡配置文件，在其中填入下面的信息：

```
[root@linuxprobe ~]# cd /etc/sysconfig/network-scripts/
[root@linuxprobe network-scripts]# vim ifcfg-ens160
TYPE=Ethernet
BOOTPROTO=static
NAME=ens160
ONBOOT=yes
IPADDR=192.168.10.10
NETMASK=255.255.255.0
GATEWAY=192.168.10.1
DNS1=192.168.10.1
```

执行重启网卡设备的命令，然后通过ping命令测试网络能否联通。由于在Linux系统中ping命令不会自动终止，因此需要手动按下Ctrl-c键来强行结束进程。

```
[root@linuxprobe network-scripts]# nmcli connection reload ens160
[root@linuxprobe network-scripts]# ping 192.168.10.10
PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
64 bytes from 192.168.10.10: icmp_seq=1 ttl=64 time=0.083 ms
64 bytes from 192.168.10.10: icmp_seq=2 ttl=64 time=0.110 ms
64 bytes from 192.168.10.10: icmp_seq=3 ttl=64 time=0.106 ms
64 bytes from 192.168.10.10: icmp_seq=4 ttl=64 time=0.035 ms
^C
--- 192.168.10.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 84ms
rtt min/avg/max/mdev = 0.035/0.083/0.110/0.031 ms
[root@linuxprobe network-scripts]# 
```

是不是感觉很有意思~当然如果这个实验失败也不用气馁，后面会有章节专门讲解，把关注点继续放回到Vim编辑器上吧。

###### **4.1.4 配置软件仓库**

本书前面讲到，软件仓库的作用是为了进一步简化RPM管理软件的难度以及自动分析所需软件包及其依赖关系的技术。可以把Yum或dnf想象成是一个硕大的软件仓库，里面保存有几乎所有常用的工具，而且只需要说出所需的软件包名称，系统就会自动为您搞定一切。

既然要使用软件仓库，就要先把它搭建起来，然后将其配置规则确定好才行。鉴于第6章才会讲解Linux的存储结构和设备挂载操作，所以当前还是将重心放到Vim编辑器的学习上。如果遇到看不懂的参数也不要紧，后面章节会单独讲解。

Yum与dnf软件仓库的配置文件是通用的，也就是说填写好来配置文件信息后，这两个命令都是可以正常使用。建议在8版本系统中使用dnf作为软件的安装命令，具备更高的效率，以及支持多线程同时安装软件的优势。

搭建并配置软件仓库的大致步骤如下所示。

**第1步**：进入到/etc/yum.repos.d/目录中（因为该目录存放着软件仓库的配置文件）。

**第2步**：使用Vim编辑器创建一个名为rhel8.repo的新配置文件（文件名称可随意，但后缀必须为.repo），逐项写入下面的配置参数并保存退出。

> 仓库名称：具有唯一性的标识名称，不应与其他软件仓库发生冲突。
>
> 描述信息（name）：可以是一些介绍词，易于识别软件仓库的用处。
>
> 仓库位置（baseurl）：即软件包的获取方式，可以使用FTP、HTTP与本地file。
>
> 是否启用（enabled）：设置此源是否被使用，1为使用，0为禁用。
>
> 是否检查（gpgcheck）：设置此源是否被校验，1为校验，0为禁用。
>
> 公钥位置（gpgkey）：若上面参数开启了校验功能，则此处为公钥文件位置。若没有开启，此省略不写。

**第3步**：按配置参数中所填写的仓库位置挂载光盘，并把光盘挂载信息写入到/etc/fstab文件中。

**第4步**：使用“dnf install httpd -y”命令检查软件仓库是否已经可用。

开始实战！

进入/etc/yum.repos.d目录中后创建软件仓库的配置文件：

```
[root@linuxprobe ~]# cd /etc/yum.repos.d/
[root@linuxprobe yum.repos.d]# vim rhel8.repo
[BaseOS]
name=BaseOS
baseurl=file:///media/cdrom/BaseOS
enabled=1
gpgcheck=0
[AppStream]
name=AppStream
baseurl=file:///media/cdrom/AppStream
enabled=1
gpgcheck=0
```

创建挂载点后进行挂载操作，并设置成开机自动挂载（详见第6章）：

```
[root@linuxprobe yum.repos.d]# mkdir -p /media/cdrom 
[root@linuxprobe yum.repos.d]# mount /dev/cdrom /media/cdrom
mount: /media/cdrom: WARNING: device write-protected, mounted read-only.
[root@linuxprobe yum.repos.d]# vim /etc/fstab
/dev/cdrom /media/cdrom iso9660 defaults 0 0
```

尝试使用软件仓库的dnf命令来安装Web服务，软件包名称叫做httpd，安装后出现**Complete!**则代表配置正确：

```
[root@linuxprobe ~]# dnf install httpd -y
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
AppStream 3.1 MB/s | 3.2 kB 00:00
BaseOS 2.7 MB/s | 2.7 kB 00:00
Dependencies resolved.
………………省略部分输出信息………………
Installed:
httpd-2.4.37-10.module+el8+2764+7127e69e.x86_64
apr-util-bdb-1.6.1-6.el8.x86_64
apr-util-openssl-1.6.1-6.el8.x86_64
apr-1.6.3-9.el8.x86_64
apr-util-1.6.1-6.el8.x86_64
httpd-filesystem-2.4.37-10.module+el8+2764+7127e69e.noarch
httpd-tools-2.4.37-10.module+el8+2764+7127e69e.x86_64
mod_http2-1.11.3-1.module+el8+2443+605475b7.x86_64
redhat-logos-httpd-80.7-1.el8.noarch

Complete!
```

对于习惯使用yum命令来安装软件的同学，不需要有压力，因为您依然可以使用yum install httpd来安装软件，只是将dnf替换成yum即可，RHEL 8版本很好的兼容了用户习惯~

##### **4.2 编写Shell脚本**

可以将Shell终端解释器当作人与计算机硬件之间的“翻译官”，它作为用户与Linux系统内部的通信媒介，除了能够支持各种变量与参数外，还提供了诸如循环、分支等高级编程语言才有的控制结构特性。要想正确使用Shell中的这些功能特性，准确下达命令尤为重要。Shell脚本命令的工作方式有两种：交互式和批处理。

> 交互式（Interactive）：用户每输入一条命令就立即执行。
>
> 批处理（Batch）：由用户事先编写好一个完整的Shell脚本，Shell会一次性执行脚本中诸多的命令。

在Shell脚本中不仅会用到前面学习过的很多Linux命令以及正则表达式、管道符、数据流重定向等语法规则，还需要把内部功能模块化后通过逻辑语句进行处理，最终形成日常所见的Shell脚本。

查看SHELL变量可以发现当前系统已经默认使用Bash作为命令行终端解释器了：

```
[root@linuxprobe ~]# echo $SHELL
/bin/bash
```

###### **4.2.1 编写简单的脚本**

估计读者在看完上文中有关Shell脚本的复杂描述后，会累觉不爱吧。但是，上文指的是一个高级Shell脚本的编写原则，其实使用Vim编辑器把Linux命令按照顺序依次写入到一个文件中，这就是一个简单的脚本了。

例如，如果想查看当前所在工作路径并列出当前目录下所有的文件及属性信息，实现这个功能的脚本应该类似于下面这样：

```
[root@linuxprobe ~]# vim example.sh
#!/bin/bash 
#For Example BY linuxprobe.com 
pwd 
ls -al
```

Shell脚本文件的名称可以任意，但为了避免被误以为是普通文件，建议将.sh后缀加上，以表示是一个脚本文件。

在上面的这个example.sh脚本中实际上出现了三种不同的元素：第一行的脚本声明（#!）用来告诉系统使用哪种Shell解释器来执行该脚本；第二行的注释信息（#）是对脚本功能和某些命令的介绍信息，使得自己或他人在日后看到这个脚本内容时，可以快速知道该脚本的作用或一些警告信息；第三、四行的可执行语句也就是我们平时执行的Linux命令了。什么？！你们不相信这么简单就编写出来了一个脚本程序，来执行一下看看结果：

```
[root@linuxprobe ~]# bash example.sh
/root
total 60
dr-xr-x---. 15 root root  4096 Oct 12 00:41 .
dr-xr-xr-x. 17 root root   224 Jul 21 05:04 ..
-rw-------.  1 root root  1407 Jul 21 05:09 anaconda-ks.cfg
-rw-------.  1 root root   335 Jul 24 06:33 .bash_history
-rw-r--r--.  1 root root    18 Aug 13  2018 .bash_logout
-rw-r--r--.  1 root root   176 Aug 13  2018 .bash_profile
………………省略部分输出信息………………
```

除了上面用bash解释器命令直接运行Shell脚本文件外，第二种运行脚本程序的方法是通过输入完整路径的方式来执行。但默认会因为权限不足而提示报错信息，此时只需要为脚本文件增加执行权限即可（详见第5章）。初次学习Linux系统的读者不用心急，等下一章学完用户身份和权限后再来做这个实验也不迟：

```
[root@linuxprobe ~]# ./example.sh
bash: ./Example.sh: Permission denied
[root@linuxprobe ~]# chmod u+x example.sh
[root@linuxprobe ~]# ./example.sh
/root
total 60
dr-xr-x---. 15 root root  4096 Oct 12 00:41 .
dr-xr-xr-x. 17 root root   224 Jul 21 05:04 ..
-rw-------.  1 root root  1407 Jul 21 05:09 anaconda-ks.cfg
-rw-------.  1 root root   335 Jul 24 06:33 .bash_history
-rw-r--r--.  1 root root    18 Aug 13  2018 .bash_logout
-rw-r--r--.  1 root root   176 Aug 13  2018 .bash_profile
………………省略部分输出信息………………
```

###### **4.2.2 接收用户的参数**

但是，像上面这样的脚本程序只能执行一些预先定义好的功能，未免太过死板了。为了让Shell脚本程序更好地满足用户的一些实时需求，以便灵活完成工作，必须要让脚本程序能够像之前执行命令时那样，接收用户输入的参数。

比如当用户执行某一个命令，加或不加参数的输出结果也是不同的：

```
[root@linuxprobe ~]# wc -l anaconda-ks.cfg 
44 anaconda-ks.cfg
[root@linuxprobe ~]# wc -c anaconda-ks.cfg 
1407 anaconda-ks.cfg
[root@linuxprobe ~]# wc -w anaconda-ks.cfg 
121 anaconda-ks.cfg
```

这意味着命令不仅要能接收用户输入的内容，还要有能力进行判断区别，根据不同的输入调用不同的功能。

其实，Linux系统中的Shell脚本语言早就考虑到了这些，已经内设了用于接收参数的变量，变量之间使用空格间隔。例如$0对应的是当前Shell脚本程序的名称，$#对应的是总共有几个参数，$*对应的是所有位置的参数值，$?对应的是显示上一次命令的执行返回值，而$1、$2、$3……则分别对应着第N个位置的参数值，如图4-15所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2015/07/Shell%E8%84%9A%E6%9C%AC%E7%A8%8B%E5%BA%8F%E4%B8%AD%E7%9A%84%E5%8F%82%E6%95%B0%E4%BD%8D%E7%BD%AE%E5%8F%98%E9%87%8F.png)

图4-15 Shell脚本程序中的参数位置变量

理论过后再来练习一下。尝试编写一个脚本程序示例，通过引用上面的变量参数来看下真实效果：

```
[root@linuxprobe ~]# vim example.sh
#!/bin/bash
echo "当前脚本名称为$0"
echo "总共有$#个参数，分别是$*。"
echo "第1个参数为$1，第5个为$5。"
[root@linuxprobe ~]# bash example.sh one two three four five six
当前脚本名称为example.sh
总共有6个参数，分别是one two three four five six。
第1个参数为one，第5个为five。
```

###### **4.2.3 判断用户的参数**

学习是一个登堂入室、由浅入深的过程。在学习完Linux命令、掌握Shell脚本语法变量和接收用户输入的信息之后，就要踏上新的高度—能够进一步处理接收到的用户参数。

在本书前面章节中讲到，系统在执行mkdir命令时会判断用户输入的信息，即判断用户指定的文件夹名称是否已经存在，如果存在则提示报错；反之则自动创建。Shell脚本中的条件测试语法可以判断表达式是否成立，若条件成立则返回数字0，否则便返回非零值。条件测试语法的执行格式如图4-16所示。切记，条件表达式两边均应有一个空格。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2015/07/%E6%B5%8B%E8%AF%95%E8%AF%AD%E5%8F%A5%E6%A0%BC%E5%BC%8F.png)

图4-16 条件测试语句的执行格式

------

按照测试对象来划分，条件测试语句可以分为4种：

> 文件测试语句；
>
> 逻辑测试语句；
>
> 整数值比较语句；
>
> 字符串比较语句。

文件测试即使用指定条件来判断文件是否存在或权限是否满足等情况的运算符，具体的参数如表4-3所示。

表4-3                          文件测试所用的参数

| 操作符 | 作用                       |
| ------ | -------------------------- |
| -d     | 测试文件是否为目录类型     |
| -e     | 测试文件是否存在           |
| -f     | 判断是否为一般文件         |
| -r     | 测试当前用户是否有权限读取 |
| -w     | 测试当前用户是否有权限写入 |
| -x     | 测试当前用户是否有权限执行 |

下面使用文件测试语句来判断/etc/fstab是否为一个目录类型的文件，然后通过Shell解释器的内设$?变量显示上一条命令执行后的返回值。如果返回值为0，则目录存在；如果返回值为非零的值，则意味着它不是目录，或这个目录不存在：

```
[root@linuxprobe ~]# [ -d /etc/fstab ]
[root@linuxprobe ~]# echo $?
1
```

再使用文件测试语句来判断/etc/fstab是否为一般文件，如果返回值为0，则代表文件存在，且为一般文件：

```
[root@linuxprobe ~]# [ -f /etc/fstab ]
[root@linuxprobe ~]# echo $?
0
```

判断与查询一定要敲两次命令吗？其实可以一次性搞定。

逻辑语句用于对测试结果进行逻辑分析，根据测试结果可实现不同的效果。例如在Shell终端中逻辑“与”的运算符号是&&，它表示当前面的命令执行成功后才会执行它后面的命令，因此可以用来判断/dev/cdrom文件是否存在，若存在则输出Exist字样。

```
[root@linuxprobe ~]# [ -e /dev/cdrom ] && echo "Exist"
Exist
```

除了逻辑“与”外，还有逻辑“或”，它在Linux系统中的运算符号为||，表示当前面的命令执行失败后才会执行它后面的命令，因此可以用来结合系统环境变量USER来判断当前登录的用户是否为非管理员身份：

```
[root@linuxprobe ~]# echo $USER
root
[root@linuxprobe ~]# [ $USER = root ] || echo "user"
[root@linuxprobe ~]# su - linuxprobe 
[linuxprobe@linuxprobe ~]$ [ $USER = root ] || echo "user"
user
```

第三种逻辑语句是“非”，在Linux系统中的运算符号是一个叹号（！），它表示把条件测试中的判断结果取相反值。也就是说，如果原本测试的结果是正确的，则将其变成错误的；原本测试错误的结果则将其变成正确的。

我们现在切换回到root管理员身份，再判断当前用户是否为一个非管理员的用户。由于判断结果因为两次否定而变成正确，因此会正常地输出预设信息：

```
[linuxprobe@linuxprobe ~]$ exit
logout
[root@linuxprobe ~]# [ ! $USER = root ] || echo "administrator"
administrator
```

叹号应该加到判断语句的前面，代表对整个的测试语句进行取反值操作，不应该写成“$USER != root”，因为!=代表的是不等于符号≠，所以虽然执行效果一样，但缺少了逻辑关系，这点还是要打好基础的。

### **Tips**

&&是逻辑“与”，只有当前面语句执行成功的时候才会执行后面的命令。

||是逻辑“或”，只有当前面语句执行失败的时候才会执行后面的命令。

!是逻辑“非”，代表对逻辑测试结果取反值，之前是正确则变错误，错误则变正确。

就技术图书的写作来讲，一般有两种套路：让读者真正搞懂技术了；让读者觉得自己搞懂技术了。因此市面上很多浅显的图书会让读者在学完之后感觉进步特别快，这基本上是作者有意为之，目的就是让您觉得“图书很有料，自己收获很大”，但是在步入工作岗位后就露出短板吃大亏。所以刘遄老师决定继续提高难度，为读者增加一个综合的示例，一方面作为前述知识的总结，另一方面帮助读者夯实基础，能够在今后工作中更灵活地使用逻辑符号。

当前我们正在登录的即为管理员用户——root。下面这个示例的执行顺序是，先判断当前登录用户的USER变量名称是否等于root，然后用逻辑运算符“非”进行取反操作，效果就变成了判断当前登录的用户是否为非管理员用户了。最后若条件成立则会根据逻辑“与”运算符输出user字样；或条件不满足则会通过逻辑“或”运算符输出root字样，而如果前面的&&不成立才会执行后面的||符号。

```
[root@linuxprobe ~]# [ ! $USER = root ] && echo "user" || echo "root"
root
```

------

整数比较运算符仅是对数字的操作，不能将数字与字符串、文件等内容一起操作，而且不能想当然地使用日常生活中的等号、大于号、小于号等来判断。因为等号与赋值命令符冲突，大于号和小于号分别与输出重定向命令符和输入重定向命令符冲突。因此一定要使用规范的整数比较运算符来进行操作。可用的整数比较运算符如表4-4所示。

表4-4                         可用的整数比较运算符

| 操作符 | 作用           |
| ------ | -------------- |
| -eq    | 是否等于       |
| -ne    | 是否不等于     |
| -gt    | 是否大于       |
| -lt    | 是否小于       |
| -le    | 是否等于或小于 |
| -ge    | 是否大于或等于 |

接下来小试牛刀。先测试一下10是否大于10以及10是否等于10（通过输出的返回值内容来判断）：

```
[root@linuxprobe ~]# [ 10 -gt 10 ]
[root@linuxprobe ~]# echo $?
1
[root@linuxprobe ~]# [ 10 -eq 10 ]
[root@linuxprobe ~]# echo $?
0
```

在2.4节曾经讲过free命令，它能够用来获取当前系统正在使用及可用的内存量信息。接下来先使用free -m命令查看内存使用量情况（单位为MB），然后通过grep Mem:命令过滤出剩余内存量的行，再用awk '{print $4}'命令只保留第四列，这个演示确实有些难度，但看懂后会觉得很有意思，没准在运维工作中也会用得上。

```
[root@linuxprobe ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1966        1374         128          16         463         397
Swap:          2047          66        1981
[root@linuxprobe ~]# free -m | grep Mem:
Mem:           1966        1374         128          16         463         397
[root@linuxprobe ~]# free -m | grep Mem: | awk '{print $4}'
128
```

如果以后想把这个命令写入到Shell脚本里，那么建议把输出结果赋值给一个变量，这样方便其他命令进行调用：

```
[root@linuxprobe ~]# FreeMem=`free -m | grep Mem: | awk '{print $4}'`
[root@linuxprobe ~]# echo $FreeMem 
128
```

上面用于获取内存可用量的命令以及步骤可能有些“超纲”了，如果不能理解领会也不用担心，接下来才是重点。我们使用整数运算符来判断内存可用量的值是否小于1024，若小于则会提示“Insufficient Memory”（内存不足）的字样：

```
[root@linuxprobe ~]# [ $FreeMem -lt 1024 ] && echo "Insufficient Memory"
Insufficient Memory
```

字符串比较语句用于判断测试字符串是否为空值，或两个字符串是否相同。它经常用来判断某个变量是否未被定义（即内容为空值），理解起来也比较简单。字符串比较中常见的运算符如表4-5所示。

表4-5                        常见的字符串比较运算符

| 操作符 | 作用                   |
| ------ | ---------------------- |
| =      | 比较字符串内容是否相同 |
| !=     | 比较字符串内容是否不同 |
| -z     | 判断字符串内容是否为空 |

接下来通过判断String变量是否为空值，进而判断是否定义了这个变量：

```
[root@linuxprobe ~]# [ -z $String ]
[root@linuxprobe ~]# echo $?
0
```

再尝试引入逻辑运算符来试一下。当用于保存当前语系的环境变量值LANG不是英语（en.US）时，则会满足逻辑测试条件并输出“Not en.US”（非英语）的字样：

```
[root@linuxprobe ~]# echo $LANG
en_US.UTF-8
[root@linuxprobe ~]# [ ! $LANG = "en.US" ] && echo "Not en.US"
Not en.US
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **4.3 流程控制语句**

尽管此时可以通过使用Linux命令、管道符、重定向以及条件测试语句来编写最基本的Shell脚本，但是这种脚本并不适用于生产环境。原因是它不能根据真实的工作需求来调整具体的执行命令，也不能根据某些条件实现自动循环执行，通俗来讲——不能根据实际情况做出调整。

通常脚本都是从上至下一股脑的执行，效率是效率，但一旦某条命令执行失败了，那后面的功能全都会受到影响。假如有一天遇到了心仪的 他（她），心中默默的规划如下：

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/Shell%E8%84%9A%E6%9C%AC%E6%B5%81%E7%A8%8B%E5%9B%BE-2.jpg)

结果可能见面聊天后就觉得不合适了，后续的一起吃饭和看电影就要停止行动，转而去做其他事情，因此需要判断语句帮助完成。

接下来我们通过if、for、while、case这4种流程控制语句来学习编写难度更大、功能更强的Shell脚本。为了保证下文的实用性和趣味性，做到寓教于乐，我会尽可能多地讲解各种不同功能的Shell脚本示例，而不是逮住一个脚本不放，在它原有内容的基础上修修补补。尽管这种修补式的示例教学也可以让读者明白理论知识，但是却无法开放思路，不利于日后的工作。

###### **4.3.1 if条件测试语句**

if条件测试语句可以让脚本根据实际情况自动执行相应的命令。从技术角度来讲，if语句分为单分支结构、双分支结构、多分支结构；其复杂度随着灵活度一起逐级上升。

if条件语句的单分支结构由if、then、fi关键词组成，而且只在条件成立后才执行预设的命令，相当于口语的“如果……那么……”。单分支的if语句属于最简单的一种条件判断结构，语法格式如图4-17所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E5%8D%95%E5%88%86%E6%94%AF%E7%9A%84if%E8%AF%AD%E5%8F%A5.png)

图4-17 单分支的if条件语句

使用单分支的if条件语句来判断/media/cdrom目录是否存在，若不存在就创建这个目录，反之则结束条件判断和整个Shell脚本。

```
[root@linuxprobe ~]# vim mkcdrom.sh
#!/bin/bash
DIR="/media/cdrom"
if [ ! -d $DIR ]
then    
        mkdir -p $DIR
fi 
```

由于第5章才讲解用户身份与权限，因此这里继续用“bash 脚本名称”的方式来执行脚本。在正常情况下，顺利执行完脚本文件后没有任何输出信息，但是可以使用ls命令验证/media/cdrom目录是否已经成功创建：

```
[root@linuxprobe ~]# bash mkcdrom.sh
[root@linuxprobe ~]# ls -ld /media/cdrom
drwxr-xr-x. 2 root root 6 Oct 13 21:34 /media/cdrom
```

------

if条件语句的双分支结构由if、then、else、fi关键词组成，它进行一次条件匹配判断，如果与条件匹配，则去执行相应的预设命令；反之则去执行不匹配时的预设命令，相当于口语的“如果……那么……或者……那么……”。if条件语句的双分支结构也是一种很简单的判断结构，语法格式如图4-18所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2015/07/%E5%8F%8C%E5%88%86%E6%94%AF%E7%BB%93%E6%9E%84-1.png)

图4-18 双分支的if条件语句

下面使用双分支的if条件语句来验证某台主机是否在线，然后根据返回值的结果，要么显示主机在线信息，要么显示主机不在线信息。这里的脚本主要使用ping命令来测试与对方主机的网络联通性，而Linux系统中的ping命令不像Windows一样尝试4次就结束，因此为了避免用户等待时间过长，需要通过-c参数来规定尝试的次数，并使用-i参数定义每个数据包的发送间隔，以及使用-W参数定义等待超时时间。

```
[root@linuxprobe ~]# vim chkhost.sh
#!/bin/bash
ping -c 3 -i 0.2 -W 3 $1 &> /dev/null
if [ $? -eq 0 ]
then
        echo "Host $1 is On-line."
else
        echo "Host $1 is Off-line."
fi
```

刚刚在4.2.3小节中用过$?变量，作用是显示上一次命令的执行返回值。若前面的那条语句成功执行，则$?变量会显示数字0，反之则显示一个非零的数字（可能为1，也可能为2，取决于系统版本）。因此可以使用整数比较运算符来判断$?变量是否为0，从而获知那条语句的最终判断情况。这里的服务器IP地址为192.168.10.10，来验证一下脚本的效果：

```
[root@linuxprobe ~]# bash chkhost.sh 192.168.10.10
Host 192.168.10.10 is On-line.
[root@linuxprobe ~]# bash chkhost.sh 192.168.10.20
Host 192.168.10.20 is Off-line.
```

if条件语句的多分支结构由if、then、else、elif、fi关键词组成，它进行多次条件匹配判断，这多次判断中的任何一项在匹配成功后都会执行相应的预设命令，相当于口语的“如果……那么……如果……那么……”。if条件语句的多分支结构是工作中最常使用的一种条件判断结构，尽管相对复杂但是更加灵活，语法格式如图4-19所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/%E5%A4%9A%E5%88%86%E6%94%AF%E7%9A%84if%E6%9D%A1%E4%BB%B6%E8%AF%AD%E5%8F%A5.png)

图 4-19 多分支的if条件语句

下面使用多分支的if条件语句来判断用户输入的分数在哪个成绩区间内，然后输出如Excellent、Pass、Fail等提示信息。在Linux系统中，read是用来读取用户输入信息的命令，能够把接收到的用户输入信息赋值给后面的指定变量，-p参数用于给予用户一些提示信息。

在下面的脚本示例中，只有当用户输入的分数大于等于85分且小于等于100分，才输出Excellent字样；若分数不满足该条件（即匹配不成功），则继续判断分数是否大于等于70分且小于等于84分，如果是，则输出Pass字样；若两次都落空（即两次的匹配操作都失败了），则输出Fail字样：

```
[root@linuxprobe ~]# vim chkscore.sh
#!/bin/bash
read -p "Enter your score（0-100）：" GRADE
if [ $GRADE -ge 85 ] && [ $GRADE -le 100 ] ; then
        echo "$GRADE is Excellent"
elif [ $GRADE -ge 70 ] && [ $GRADE -le 84 ] ; then
        echo "$GRADE is Pass"
else
        echo "$GRADE is Fail" 
fi
[root@linuxprobe ~]# bash chkscore.sh
Enter your score（0-100）：88
88 is Excellent
[root@linuxprobe ~]# bash chkscore.sh 
Enter your score（0-100）：80
80 is Pass
```

下面执行该脚本。当用户输入的分数分别为30和200时，其结果如下：

```
[root@linuxprobe ~]# bash chkscore.sh  
Enter your score（0-100）：30
30 is Fail
[root@linuxprobe ~]# bash chkscore.sh
Enter your score（0-100）：200 
200 is Fail
```

为什么输入的分数为200时，依然显示Fail呢？原因很简单—没有成功匹配脚本中的两个条件判断语句，因此自动执行了最终的兜底策略。可见，这个脚本还不是很完美，建议读者自行完善这个脚本，使得用户在输入大于100或小于0的分数时，给予Error报错字样的提示。

###### **4.3.2 for条件循环语句**

for循环语句允许脚本一次性读取多个信息，然后逐一对信息进行操作处理，当要处理的数据有范围时，使用for循环语句再适合不过了。for循环语句的语法格式如图4-20所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2015/07/for%E6%9D%A1%E4%BB%B6%E8%AF%AD%E5%8F%A5-1.png)

图4-20 for范围循环语句

下面使用for循环语句从列表文件中读取多个用户名，然后为其逐一创建用户账户并设置密码。首先创建用户名称的列表文件users.txt，每个用户名称单独一行。读者可以自行决定具体的用户名称和个数：

```
[root@linuxprobe ~]# vim users.txt
andy
barry
carl
duke
eric
george
```

接下来编写Shell脚本addusers.sh。在脚本中使用read命令读取用户输入的密码值，然后赋值给PASSWD变量，通过-p参数向用户显示一段提示信息，告诉用户正在输入的内容即将作为账户密码。在执行该脚本后，会自动使用从列表文件users.txt中获取到所有的用户名称，然后逐一使用“id 用户名”命令查看用户的信息，并使用$?判断这条命令是否执行成功，也就是判断该用户是否已经存在。

```
[root@linuxprobe ~]# vim addusers.sh
#!/bin/bash
read -p "Enter The Users Password : " PASSWD
for UNAME in `cat users.txt`
do
        id $UNAME &> /dev/null
        if [ $? -eq 0 ]
        then
                echo "$UNAME , Already exists"
        else
                useradd $UNAME &> /dev/null
                echo "$PASSWD" | passwd --stdin $UNAME &> /dev/null
                echo "$UNAME , Create success"
        fi
done
```

### **Tips**

/dev/null是一个被称作Linux黑洞的文件，把输出信息重定向到这个文件等同于删除数据（类似于没有回收功能的垃圾箱），可以让用户的屏幕窗口保持简洁。

执行批量创建用户的Shell脚本addusers.sh，在输入为账户设定的密码后将由脚本自动检查并创建这些账户。由于已经将多余的信息通过输出重定向符转移到了/dev/null黑洞文件中，因此在正常情况下屏幕窗口除了“用户账户创建成功”（Create success）的提示后不会有其他内容。

在Linux系统中，/etc/passwd是用来保存用户账户信息的文件。如果想确认这个脚本是否成功创建了用户账户，可以打开这个文件，看其中是否有这些新创建的用户信息。

```
[root@linuxprobe ~]# bash addusers.sh
Enter The Users Password : linuxprobe
andy , Create success
barry , Create success
carl , Create success
duke , Create success
eric , Create success
george , Create success
[root@linuxprobe ~]# tail -6 /etc/passwd
andy:x:1001:1001::/home/andy:/bin/bash
barry:x:1002:1002::/home/barry:/bin/bash
carl:x:1003:1003::/home/carl:/bin/bash
duke:x:1004:1004::/home/duke:/bin/bash
eric:x:1005:1005::/home/eric:/bin/bash
george:x:1006:1006::/home/george:/bin/bash
```

您还记得在学习双分支if条件语句时，用到的那个测试主机是否在线的脚本么？既然我们现在已经掌握了for循环语句，不妨做些更酷的事情，比如尝试让脚本从文本中自动读取主机列表，然后自动逐个测试这些主机是否在线。

首先创建一个主机列表文件ipaddrs.txt：

```
[root@linuxprobe ~]# vim ipaddrs.txt
192.168.10.10
192.168.10.11
192.168.10.12
```

然后前面的双分支if条件语句与for循环语句相结合，让脚本从主机列表文件ipaddrs.txt中自动读取IP地址（用来表示主机）并将其赋值给HLIST变量，从而通过判断ping命令执行后的返回值来逐个测试主机是否在线。脚本中出现的$（命令）是一种完全类似于第3章的转义字符中反引号`命令`的Shell操作符，效果同样是执行括号或双引号括起来的字符串中的命令。大家在编写脚本时，多学习几种类似的新方法，可在工作中大显身手：

```
[root@linuxprobe ~]# vim CheckHosts.sh
#!/bin/bash
HLIST=$(cat ~/ipaddrs.txt)
for IP in $HLIST
do
        ping -c 3 -i 0.2 -W 3 $IP &> /dev/null
        if [ $? -eq 0 ]  
        then
                echo "Host $IP is On-line."
        else
                echo "Host $IP is Off-line."
        fi
done
[root@linuxprobe ~]# ./CheckHosts.sh
Host 192.168.10.10 is On-line.
Host 192.168.10.11 is Off-line.
Host 192.168.10.12 is Off-line.
```

细心的读者肯定发现了，Shell脚本中的代码缩进格式会根据不同的语句而改变，这个是由Vim编辑器自动完成的，用户无需额外操作。但如果您使用的是7版本以前的系统则没有这个自动缩进功能，但功能是不受影响的，只是影响了阅读体验而已。

###### **4.3.3 while条件循环语句**

while条件循环语句是一种让脚本根据某些条件来重复执行命令的语句，它的循环结构往往在执行前并不确定最终执行的次数，完全不同于for循环语句中有目标、有范围的使用场景。while循环语句通过判断条件测试的真假来决定是否继续执行命令，若条件为真就继续执行，为假就结束循环。while语句的语法格式如图4-21所示。![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2015/07/while%E6%9D%A1%E4%BB%B6%E8%AF%AD%E5%8F%A5-1.png)

图4-21 while条件循环语句

接下来结合使用多分支的if条件测试语句与while条件循环语句，编写一个用来猜测数值大小的脚本Guess.sh。该脚本使用$RANDOM变量来调取出一个随机的数值（范围为0～32767），将这个随机数对1000进行取余操作，并使用expr命令取得其结果，再用这个数值与用户通过read命令输入的数值进行比较判断。这个判断语句分为三种情况，分别是判断用户输入的数值是等于、大于还是小于使用expr命令取得的数值。当前，现在这些内容不是重点，我们当前要关注的是while条件循环语句中的条件测试始终为true，因此判断语句会无限执行下去，直到用户输入的数值等于expr命令取得的数值后，这两者相等之后才运行exit 0命令，终止脚本的执行。

```
[root@linuxprobe ~]# vim Guess.sh
#!/bin/bash
PRICE=$(expr $RANDOM % 1000)
TIMES=0
echo "商品实际价格为0-999之间，猜猜看是多少？"
while true
do
        read -p "请输入您猜测的价格数目：" INT
        let TIMES++
        if [ $INT -eq $PRICE ] ; then
                echo "恭喜您答对了，实际价格是 $PRICE"
                echo "您总共猜测了 $TIMES 次"
                exit
        elif [ $INT -gt $PRICE ] ; then
                echo "太高了！"
        else
                echo "太低了！"
        fi
done
```

在这个Guess.sh脚本中，我们添加了一些交互式的信息，从而使得用户与系统的互动性得以增强。而且每当循环到let TIMES++命令时都会让TIMES变量内的数值加1，用来统计循环总计执行了多少次。这可以让用户得知总共猜测了多少次之后，才猜对价格。

```
[root@linuxprobe ~]# bash Guess.sh
商品实际价格为0-999之间，猜猜看是多少？
请输入您猜测的价格数目：500
太低了！
请输入您猜测的价格数目：800
太高了！
请输入您猜测的价格数目：650
太低了！
请输入您猜测的价格数目：720
太高了！
请输入您猜测的价格数目：690
太低了！
请输入您猜测的价格数目：700
太高了！
请输入您猜测的价格数目：695
太高了！
请输入您猜测的价格数目：692
太高了！
请输入您猜测的价格数目：691
恭喜您答对了，实际价格是 691
您总共猜测了 9 次
```

当条件为true（真）的时候，while语句会一直循环下去，只有碰到exit才会结束，所以同学们一定要记得加上哦。

###### **4.3.4 case条件测试语句**

如果您之前学习过C语言，看到这一小节的标题肯定会会心一笑“这不就是switch语句嘛！”是的，case条件测试语句和switch语句的功能非常相似！case语句是在多个范围内匹配数据，若匹配成功则执行相关命令并结束整个条件测试；而如果数据不在所列出的范围内，则会去执行星号（*）中所定义的默认命令。case语句的语法结构如图4-22所示。

![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2020/10/case%E6%9D%A1%E4%BB%B6%E6%B5%8B%E8%AF%95%E8%AF%AD%E5%8F%A5%E7%9A%84%E8%AF%AD%E6%B3%95%E7%BB%93%E6%9E%84.png)

图4-22 case条件测试语句

在前文介绍的Guess.sh脚本中有一个致命的弱点—只能接受数字！您可以尝试输入一个字母，会发现脚本立即就崩溃了。原因是字母无法与数字进行大小比较，例如，“a是否大于等于3”这样的命题是完全错误的。必须有一定的措施来判断用户的输入内容，当用户输入的内容不是数字时，脚本能予以提示，从而免于崩溃。

通过在脚本中组合使用case条件测试语句和通配符（详见第3章），完全可以满足这里的需求。接下来我们编写脚本Checkkeys.sh，提示用户输入一个字符并将其赋值给变量KEY，然后根据变量KEY的值向用户显示其值是字母、数字还是其他字符。

```
[root@linuxprobe ~]# vim Checkkeys.sh
#!/bin/bash
read -p "请输入一个字符，并按Enter键确认：" KEY
case "$KEY" in
        [a-z]|[A-Z])
                echo "您输入的是 字母。"
                ;;
        [0-9])
                echo "您输入的是 数字。"
                ;;
        *)
                echo "您输入的是 空格、功能键或其他控制字符。"
esac
[root@linuxprobe ~]# bash Checkkeys.sh
请输入一个字符，并按Enter键确认：6
您输入的是 数字。
[root@linuxprobe ~]# bash Checkkeys.sh
请输入一个字符，并按Enter键确认：p
您输入的是 字母。
[root@linuxprobe ~]# bash Checkkeys.sh
请输入一个字符，并按Enter键确认：^[[15~
您输入的是 空格、功能键或其他控制字符。
```

##### **4.4 计划任务服务程序**

经验丰富的系统运维工程师可以使得Linux在无需人为介入的情况下，在指定的时间段自动启用或停止某些服务或命令，从而实现运维的自动化。尽管我们现在已经有了功能彪悍的脚本程序来执行一些批处理工作，但是，如果仍然需要在每天凌晨两点敲击键盘回车键来执行这个脚本程序，这简直太痛苦了（当然，也可以训练您的小猫在半夜按下回车键）。接下来，刘遄老师将向大家讲解如何设置服务器的计划任务服务，把周期性、规律性的工作交给系统自动完成。

计划任务分为一次性计划任务与长期性计划任务，大家可以按照如下方式理解。

> 一次性计划任务：今晚23点30分重启网站服务。
>
> 长期性计划任务：每周一的凌晨3点25分都把/home/wwwroot目录打包备份为backup.tar.gz。

顾名思义，一次性计划任务只执行一次，一般用于临时的工作需求。用at命令实现这种功能，只需要写成“at 时间”的形式就行。如果想要查看已设置好但还未执行的一次性计划任务，可以使用“at -l”命令；要想将其删除，可以用“atrm 任务序号”，at命令的参数及其作用如表4-6所示。

表4-6                        at命令的参数及其作用

| 参数 | 作用                   |
| ---- | ---------------------- |
| -f   | 指定包含命令的任务文件 |
| -q   | 指定新任务名称         |
| -l   | 显示待执行任务列表     |
| -d   | 删除指定待执行任务     |
| -m   | 任务执行后给用户发邮件 |

在使用at命令来设置一次性计划任务时，默认采用的是交互式方法。例如，使用下述命令将系统设置为在今晚23:30分自动重启网站服务。

```
[root@linuxprobe ~]# at 23:30
warning: commands will be executed using /bin/sh
at> systemctl restart httpd
at> 此处请同时按下<Ctrl>+<d>键来结束编写计划任务
job 1 at Wed Oct 14 23:30:00 2020
[root@linuxprobe ~]# at -l
1 Wed Oct 14 23:30:00 2020 a root
```

看到warning提醒信息不要慌，at命令只是在告诉我们接下来的任务将由sh解释器负责执行，这与此前学习的bash解释器基本一致，不需要有额外操作。

另外如果读者想挑战一下难度更大但简捷性更高的方式，可以把前面学习的管道符（任意门）放到两条命令之间，让at命令接收前面echo命令的输出信息，以达到通过非交互式的方式创建计划一次性任务的目的。

```
[root@linuxprobe ~]# echo "systemctl restart httpd" | at 23:30
warning: commands will be executed using /bin/sh
job 2 at Wed Oct 14 23:30:00 2020
[root@linuxprobe ~]# at -l
1 Wed Oct 14 23:30:00 2020 a root
2 Wed Oct 14 23:30:00 2020 a root
```

上面设置了两条一样的计划任务，使用atrm命令轻松删除其中一个：

```
[root@linuxprobe ~]# atrm 2
[root@linuxprobe ~]# at -l
1 Wed Oct 14 23:30:00 2020 a root
```

还有种特殊场景——把计划任务写入到Shell脚本中，当用户激活后再开始倒计时执行，而不是像上面那样“at 23:30”固定的时间，该怎么办呢？

一般会使用“at now +2 MINUTE”的方式操作，这样代表2分钟（MINUTE）后执行这个任务，也可以替代成小时（HOUR）、日（DAY）、月（MONTH）等词汇：

```
[root@linuxprobe ~]# at now +2 MINUTE
warning: commands will be executed using /bin/sh
at> systemctl restart httpd
at> 此处请同时按下<Ctrl>+<d>键来结束编写计划任务
job 3 at Wed Oct 14 22:50:00 2020
```

------

还有些时候，我们会希望Linux系统能够周期性地、有规律地执行某些具体的任务，那么Linux系统中默认启用的crond服务简直再适合不过了。创建、编辑计划任务的命令为“crontab -e”，查看当前计划任务的命令为“crontab -l”，删除某条计划任务的命令为“crontab -r”。另外，如果您是以管理员的身份登录的系统，还可以在crontab命令中加上-u参数来编辑他人的计划任务。crontab命令的参数及其作用如表4-7所示。

表4-7                        crontab命令的参数及其作用

| 参数 | 作用         |
| ---- | ------------ |
| -e   | 编辑计划任务 |
| -u   | 指定用户名称 |
| -l   | 列出任务列表 |
| -r   | 删除计划任务 |

在正式部署计划任务前，请先跟刘遄老师念一下口诀“分、时、日、月、星期 命令”。这是使用crond服务设置任务的参数格式（其格式见表4-8）。需要注意的是，如果有些字段没有被设置，则需要使用星号（*****）占位，如图4-23所示。![第4章 Vim编辑器与Shell命令脚本第4章 Vim编辑器与Shell命令脚本](https://www.linuxprobe.com/wp-content/uploads/2015/02/cron%E8%AE%A1%E5%88%92%E4%BB%BB%E5%8A%A1%E7%9A%84%E5%8F%82%E6%95%B0.png)

图4-23 使用crond设置任务的参数格式

表4-8                    使用crond设置任务的参数字段说明

| 字段 | 说明                                     |
| ---- | ---------------------------------------- |
| 分钟 | 取值为0～59的整数                        |
| 小时 | 取值为0～23的任意整数                    |
| 日期 | 取值为1～31的任意整数                    |
| 月份 | 取值为1～12的任意整数                    |
| 星期 | 取值为0～7的任意整数，其中0与7均为星期日 |
| 命令 | 要执行的命令或程序脚本                   |

假设在每周一、三、五的凌晨3点25分，都需要使用tar命令把某个网站的数据目录进行打包处理，使其作为一个备份文件。我们可以使用crontab -e命令来创建计划任务，为自己创建计划任务无需使用-u参数，具体的实现效果的参数如crontab -l命令结果所示：

```
[root@linuxprobe ~]# crontab -e
no crontab for root - using an empty one
crontab: installing new crontab
[root@linuxprobe ~]# crontab -l
25 3 * * 1,3,5 /usr/bin/tar -czvf backup.tar.gz /home/wwwroot
```

需要说明的是，除了用逗号（,）来分别表示多个时间段，例如“8,9,12”表示8月、9月和12月。还可以用减号（-）来表示一段连续的时间周期（例如字段“日”的取值为“12-15”，则表示每月的12～15日）。以及用除号（/）表示执行任务的间隔时间（例如“*/2”表示每隔2分钟执行一次任务）。

如果在crond服务中需要同时包含多条计划任务的命令语句，应每行仅写一条。例如再添加一条计划任务，它的功能是每周一至周五的凌晨1点钟自动清空/tmp目录内的所有文件。尤其需要注意的是，在crond服务的计划任务参数中，所有命令一定要用绝对路径的方式来写，如果不知道绝对路径，请用whereis命令进行查询，rm命令路径为下面输出信息中加粗部分。

```
[root@linuxprobe ~]# whereis rm
rm: /usr/bin/rm /usr/share/man/man1/rm.1.gz /usr/share/man/man1p/rm.1p.gz
[root@linuxprobe ~]# crontab -e
crontab: installing new crontab
[root@linuxprobe ~]# crontab -l
25 3 * * 1,3,5 /usr/bin/tar -czvf backup.tar.gz /home/wwwroot
0 1 * * 1-5 /usr/bin/rm -rf /tmp/*
```

总结一下使用计划服务的注意事项——在crond服务的配置参数中，一般会像Shell脚本那样以#号开头写上注释信息，这样在日后回顾这段命令代码时可以快速了解其功能、需求以及编写人员等重要信息。另外，计划任务中的“分”字段必须有数值，绝对不能为空或是*号，而“日”和“星期”字段不能同时使用，否则就会发生冲突。

删除crond计划任务则非常简单，使用crontab -e命令进入到编辑界面中，删除里面的文本信息即可，也可以使用crontab -r命令直接进行删除：

```
[root@linuxprobe ~]# crontab -r
[root@linuxprobe ~]# crontab -l
no crontab for root
```

