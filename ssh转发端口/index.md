# ssh转发端口


本地端口转发命令：

`ssh -L <local port>:<remote host>:<remote port> <SSH hostname>`

例如，需要把本地8080端口转发到172.18.247.180这台机器上的8888端口，只需要执行下面的命令即可：

`ssh -L 8080:172.18.247.180:8888 172.18.247.180`

这个命令可以稍微优化一下：

`ssh -CfNg -L 8081:172.18.247.180:8888 172.18.247.180`

但是这样每次都要去记忆IP和端口也不是很方便，所以可以考虑使用脚本来完成，下面就是一个我临时写的脚本，命令为forward.sh：

`#!/bin/bash
if [ $1 == "start" ]; then
    echo "port forwarding starting..."
    ssh -CfNg -L 8081:172.18.247.180:8888 172.18.247.180
    ssh -CfNg -L 8082:172.18.247.181:8888 172.18.247.181
elif [ $1 == "stop" ]; then
    echo "stop port forwards"
    ssh_pids=$(ps -ef | grep -E 'ssh\ -CfNg\ -L|ssh-agent\ -l' | awk '{print $2}')
    echo ${ssh_pids}
    kill ${ssh_pids}
    echo "port forward had stopped"
else
    echo "port forwarding starting..."
    ssh -CfNg -L $1:$2:$3 $4
fi
ps -ef | grep ssh | grep -v grep`

可以将常用的转发规则直接写进脚本中，需要打开转发的时候，运行脚本：

`sh forward.sh start`

即可开始所有的转发规则，需要停止就运行：

`sh forward.sh stop`

停止所有转发。如果有临时的转发规则，则可以执行：

`sh forward.sh <local port> <remote host> <remote port> <SSH hostname>`
