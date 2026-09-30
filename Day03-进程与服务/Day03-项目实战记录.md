[day3.txt](https://github.com/user-attachments/files/32850678/day3.txt)
Day 3：Linux 进程与服务管理
一、实验环境
操作系统：Rocky Linux 9
虚拟机：VMware
主机名：localhost
用户：z
学习日期：2026-09-10

二、今日学习目标
理解 Linux 中的进程
掌握 PID 的基本概念
使用 ps 查看进程
使用 kill 终止进程
理解前台进程与后台进程
使用 & 将程序放到后台运行
使用 jobs 查看当前 Shell 的后台任务
使用 systemctl 管理系统服务
理解 active 与 enabled 的区别
使用 journalctl 查看服务日志
完成一次完整的服务故障排查

三、进程基础
1. 使用 ps 查看进程
执行：
ps

观察到当前 Shell 中存在：
PID    TTY         TIME        CMD
6399   pts/0    00:00:00     bash
10138  pts/0    00:00:00    ps

理解
bash 本身也是一个正在运行的进程。
执行 ps 时，Linux 会启动一个短暂的 ps 进程来完成进程信息查询。
因此：Linux 中运行的程序通常都会以进程的形式存在。

四、使用 PID 定位进程
执行：sleep 300
为了方便观察，在另一个终端执行：ps aux | grep sleep
发现： z    10331 ... sleep 300
          z    10350 ... grep --color=auto sleep

其中：10331 → sleep 进程的 PID
         10350 → grep 自己的 PID

五、使用 kill 终止进程
执行：kill 10331
再次执行：ps aux | grep sleep
结果只剩下：grep --color=auto sleep
说明：PID 为 10331 的 sleep 300 进程已经结束。

遇到的问题
之前尝试：kill 10238
出现：bash: kill: (10238): 没有那个进程
原因：PID 10238 对应的进程已经结束，所以 Linux 找不到这个 PID。
由此认识到：PID 是某个具体进程当前的编号，并不是某个程序固定不变的编号。

六、前台进程与后台进程
执行：sleep 300 &
得到：[1] 10666
这里：[1] → 当前 Shell 的后台任务编号
         10666 → Linux 分配给 sleep 进程的 PID
然后执行：ps aux | grep sleep
看到： z    10666 ... sleep 300
          z    10673 ... grep --color=auto sleep
确认 sleep 300 正在后台运行。

前台与后台区别
前台
sleep 300
Shell 会等待 sleep 执行结束。
后台
sleep 300 &
Shell 不需要等待 sleep 结束，可以继续执行其他命令。

七、使用 jobs 查看后台任务
执行：jobs
期间第一次误输入：jibs
Bash 提示：bash: jibs: 未找到命令
                  相似命令是：'jobs'
修正后执行：jobs
发现：[1]+  已完成    sleep 300
说明：后台的 sleep 300 已经自然运行结束。
之后再次执行：kill 10666
得到：bash: kill: (10666): 没有那个进程
原因同样是：sleep 300 已经结束，PID 10666 已经不存在。

八、systemctl 服务管理
本次实验选择 Rocky Linux 中的 OpenSSH 服务：sshd.service
使用：systemctl status sshd
查看服务状态。
正常运行时：Active: active (running)
同时看到：Main PID: 1108 (sshd)
说明 sshd 服务当前正在运行，并且有对应的主进程。

九、理解 active 和 enabled
在 systemctl status sshd 中看到：Active: active (running)
                                                 Loaded: loaded (...; enabled)
理解：
active
表示：服务现在是否正在运行。
enabled
表示：系统启动时是否设置为自动启动。
可以理解为：当前正在运行，并且设置了开机自动启动

十、模拟 SSH 服务故障
为了模拟真实运维中的服务故障，主动停止 SSH 服务：sudo systemctl stop sshd\
然后：systemctl status sshd
发现：Active: inactive (dead)
说明：sshd 服务已经停止。
同时：Main PID: 11105 (code=exited, status=0/SUCCESS)
说明原来的 sshd 主进程已经退出。

十一、使用 ps 确认进程状态
执行：ps aux | grep sshd
得到：z    11505 ... grep --color=auto sshd
这里并没有真正的：sshd 进程。
看到的实际上是：grep --color=auto sshd
也就是当前执行的 grep 命令自己。

重要经验
不能因为：ps aux | grep sshd
输出中出现了 sshd 字符串，就直接认为 sshd 进程存在。
必须看清楚：真正运行的程序到底是谁。

十二、使用 journalctl 查看服务日志
执行：journalctl -u sshd
查看 SSH 服务相关日志。
发现：Stopping OpenSSH server daemon...
         Received signal 15; terminating.
         sshd.service: Deactivated successfully.
         Stopped OpenSSH server daemon.

结合实验过程，可以判断：SSH 服务是因为执行 systemctl stop sshd 被主动停止的。
其中：signal 15
对应：SIGTERM
表示请求进程正常终止。

十三、恢复 SSH 服务
执行：sudo systemctl start sshd
然后：systemctl status sshd
恢复为：Active: active (running)
同时发现新的：Main PID: 11105 (sshd)
与停止前的 PID 不同。
说明：原来的 sshd 进程已经结束，重新启动服务后产生了新的 sshd 进程，因此 PID 发生变化。

十四、限制日志数量
执行：journalctl -u sshd -n 5
只查看最后 5 条 SSH 服务日志。
得到类似：Stopped OpenSSH server daemon.
                Starting OpenSSH server daemon...
                Server listening on 0.0.0.0 port 22.
                Server listening on :: port 22.
                Started OpenSSH server daemon.

可以判断：
SSH 服务停止
SSH 服务重新启动
开始监听 22 端口
SSH 服务启动成功

十五、完整故障排查流程
本次实验最终形成了一个基本的 Linux 服务排障思路：
发现服务异常
      ↓
systemctl status sshd
      ↓
确认服务状态
      ↓
ps aux | grep sshd
      ↓
确认相关进程是否存在
      ↓
journalctl -u sshd -n 5
      ↓
查看服务日志
      ↓
分析停止/异常原因
      ↓
systemctl start sshd
      ↓
再次检查服务状态
      ↓
确认恢复

十六、今日总结
今天最大的收获不是记住几个命令，而是开始建立：
“看到问题 → 获取信息 → 分析原因 → 执行处理 → 验证结果”的排障思维。

目前掌握的主要工具：
ps
ps aux
grep
kill
jobs
systemctl status
systemctl start
systemctl stop
journalctl

其中最重要的三个排障工具：
ps
↓
看进程

systemctl
↓
看/管理服务

journalctl
↓
看服务日志

十七、Day 3 实验结论
通过本次实验，我能够：
查看 Linux 当前运行的进程
根据 PID 定位进程
使用 kill 终止进程
区分前台和后台进程
使用 & 启动后台任务
使用 jobs 查看 Shell 后台任务
使用 systemctl 查看、停止和启动服务
区分 active 和 enabled
使用 journalctl 查看服务日志
根据日志判断服务停止原因
验证服务恢复结果
初步建立 Linux 服务故障排查思路
