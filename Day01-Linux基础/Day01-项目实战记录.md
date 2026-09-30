[day1.txt](https://github.com/user-attachments/files/32849811/day1.txt)
# Day 1：Linux 基本目录与文件操作

## 1. 实验环境

- 操作系统：Rocky Linux 9
- 虚拟机名称：rocky9
- 虚拟化环境：VMware
- SSH：本次实验未使用

---

## 2. 实验目标

熟悉 Linux 的基本目录和文件操作，不再只知道命令是什么意思，而是能够自己创建、复制、移动和删除文件。

---

## 3. 本次使用的命令

```bash
pwd
ls
cd
mkdir
touch
cp
mv
rm
ls -R

命令         作用         
------- | ---------- 
pwd      查看当前所在目录   
ls          查看目录内容     
cd         切换目录       
mkdir   创建目录       
touch   创建文件       
cp        复制文件       
mv       移动文件       
rm       删除文件       
ls -R     递归查看目录及其内容 

4. 实验过程
创建实验目录：
mkdir  project
cd  project

创建三个目录：
mkdir source1
mkdir backup1
mkdir logs1

进入 source1：
cd  source1

创建三个测试文件：
touch  {app,config,test}.txt

检查文件：
ls

得到：
app.txt
config.txt
test.txt

将 app.txt 复制到 backup1：
cp app.txt ../backup1/

检查复制结果：
ls ../backup1/

将 config.txt 移动到 logs1：
mv config.txt ../logs1/

删除 test.txt：
rm test.txt

最后检查整个项目目录：
ls -R ~/linux-lab/project

最终结果符合实验要求：
project/
├── backup1/
│   └── app.txt
├── logs1/
│   └── config.txt
└── source1/
    └── app.txt

5. 遇到的问题
刚开始没有理解：
./
../

6. 问题解决
通过查阅资料了解到：
./  表示当前目录
../ 表示当前目录的上一级目录（父目录）

例如：
cp app.txt ../backup1/
表示将当前目录中的 app.txt 复制到上一级目录中的 backup1。
通过实际操作后，对相对路径有了更加直观的理解。

7. 故障排查思路
本次实验还进行了一个简单的故障排查思考：
如果发现 test.txt 不见了，首先应该确认自己当前所在的位置：
pwd

然后检查当前目录：
ls

如果仍然不知道文件在哪里，可以进一步使用文件搜索工具进行查找，例如：
find ~/linux-lab -name "test.txt"
通过这个过程开始建立“先确认位置，再检查目录，再进一步定位”的排查思路。

8. 本次收获
之前学习这些 Linux 基础命令时，更多只是知道每个命令是什么意思。
经过今天的实际操作后，开始能够把这些基础命令串联起来使用：
pwd
 ↓
ls
 ↓
cd
 ↓
mkdir / touch
 ↓
cp / mv / rm
 ↓
ls 检查结果
最大的收获不是记住了多少条命令，而是开始知道如何实际操作，并且在遇到问题时开始思考应该如何排查。
