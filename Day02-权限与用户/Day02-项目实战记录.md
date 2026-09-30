[day2.txt](https://github.com/user-attachments/files/32850002/day2.txt)
Day 2：Linux 文件权限与用户权限管理
1. 实验环境
操作系统：Rocky Linux 9
虚拟机名称：rocky9
当前用户：z
SSH：未使用

2. 实验目标
学习 Linux 权限管理机制，理解：
Linux 文件权限结构
r/w/x 三种权限含义
数字权限表示方法
chmod 修改权限
文件权限和目录权限区别
用户与文件归属关系
Permission denied 故障排查方法

3. Linux 文件权限基础
通过：
ls -l
查看文件权限。
示例：
-rw-r--r--
权限结构：
- | rw- | r-- | r--
    |     |     |
    |     |     └── 其他用户权限
    |     └──────── 所属组权限
    └────────────── 文件所有者权限
权限含义
权限 |   含义         数字 
-- | ---------- | -- |
r     read 读取        4  
w    write 写入       2  
x     execute 执行   1  
-      无权限            0  
权限数字由三组权限相加得到：rw-     4+2+0=6

4. chmod 权限修改实验
创建测试文件 :  touch test.txt
查看默认权限 ：ls -l test.txt
结果：-rw-r--r--
表示：
  rw     所有者：可读、可写
  r--    同组用户：只读
  r--     其他用户：只读

修改权限
执行：chmod 600 test.txt
查看：ls -l test.txt
结果：-rw-------
   解释：600
   所有者：rw-
   同组用户：---
   其他用户：---
表示：只有文件所有者可以读取和修改。

5. Permission denied 故障模拟
创建文件：touch secret.txt
修改权限：chmod 000 secret.txt
查看：ls -l secret.txt
结果：----------
尝试读取：cat secret.txt
出现：权限不够

6. Permission denied 故障排查
第一步：确认当前用户
命令：whoami
结果：z
确认当前操作用户。

第二步：查看文件权限
命令：ls -l secret.txt
发现：----------
说明：文件没有任何读取权限。

第三步：分析问题
cat 命令读取文件需要：r(read)权限。
但是当前文件权限：000  所有用户均无读取权限。

第四步：修复
执行：chmod 600 secret.txt
恢复：-rw-------  
之后文件可以正常读取。

7. 文件权限与目录权限区别实验
创建目录：mkdir dir-test
默认：drwxr-xr-x
修改：chmod 600 dir-test
结果：drw-------
尝试进入：cd dir-test   
失败

原因分析
文件：x = 执行文件
目录：x = 进入目录
目录缺少 x 权限,即使目录所有者也无法进入

8. 用户权限访问案例
创建新用户：sudo useradd testuser
查看：id testuser

创建测试文件:  touch user-test.txt
查看：ls -l user-test.txt
结果：-rw-r--r--

问题
切换用户：su - testuser
尝试读取：cat /home/z/linux-lab/day2/user-test.txt
失败。

9. 深入排查：路径权限问题
文件权限：-rw-r--r--
理论上允许其他用户读取。
继续检查完整路径：namei -l /home/z/linux-lab/day2/user-test.txt
发现：/home/z        drwx------
原因：/home/z 目录权限为：700
拆解：rwx ---
只有用户 z 可以访问。

testuser
属于其他用户:  ---
没有 x 权限,因此无法进入目录。

10. 问题解决
修改：chmod 711 /home/z
权限：rwx --x --x
含义：所有者：可以查看,可以修改,可以进入
其他用户：可以进入,不能查看目录,内容不能修改
再次测试：cat /home/z/linux-lab/day2/user-test.txt
访问成功。

11. 今日故障排查总结
遇到：Permission denied
排查流程：
1. whoami
确认当前用户
    ↓
2. ls -l
查看文件权限
    ↓
3. 查看 owner/group
    ↓
4. 检查目录权限
    ↓
5. namei -l
检查完整路径
    ↓
6. 根据原因：
chmod
chown
chgrp

12. 今日收获
今天最大的收获：
以前：只知道 chmod、600、755 这些命令和数字。
今天：开始理解 Linux 权限判断逻辑。
通过实际实验：
修改权限
制造 Permission denied
分析错误原因
修复访问问题
建立了 Linux 运维中的权限排查思维。
核心理解：Linux 判断访问权限，不只看文件本身，还要看访问路径上的所有目录权限。
