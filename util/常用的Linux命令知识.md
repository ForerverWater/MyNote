```shell
# 关机
shutdown -h now

# 重启
shutdown -r now

# 查看系统内核信息
uname -a

# 查看系统内核版本
cat /proc/version

# 查看当前用户环境变量
env

cat /proc/cpuinfo

# 查看有几个逻辑cpu, 包括cpu型号
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c

# 查看有几颗cpu,每颗分别是几核
cat /proc/cpuinfo | grep physical | uniq -c

# 查看当前CPU运行在32bit还是64bit模式下, 如果是运行在32bit下也不代表CPU不支持64bit
getconf LONG_BIT

# 结果大于0, 说明支持64bit计算. lm指long mode, 支持lm则是64bit
cat /proc/cpuinfo | grep flags | grep ' lm ' | wc -l
```

```shell
# 建立软连接
ln -s /usr/local/jdk1.8/ jdk

# rpm相关
rpm -qa | grep 软件名

# 在各个用户的.bash_profile中添加重命名配置
alias ll='ls -alF'

# 后台运行,并且有nohup.out输出
nohup xxx &

# 后台运行, 不输出任何日志
nohup xxx > /dev/null &

# 后台运行, 并将错误信息做标准输出到日志中 
nohup xxx >out.log 2>&1 &

# 查看命令路径
which <命令>

# 查看进程所有打开最大fd数
ulimit -n

# 最近登录的5个账号
last -n 5

# 设计固定ip
ifconfig em1  192.168.5.177 netmask 255.255.255.0

# 查看进程内加载的环境变量
# 也可以去 cd /proc 目录下, 查看进程内存中加载的东西
ps eww -p  XXXXX(进程号)

# 添加用户，配置sudo权限
useradd 用户名
passwd 用户名

#增加sudo权限
vim /etc/sudoers

# 强制关闭进程名包含xxx的所有进程
ps aux|grep xxx | grep -v grep | awk '{print $2}' | xargs kill -9
```

查看磁盘, 文件目录基本信息
```shell
# 查看磁盘挂载情况
mount

# 查看磁盘分区信息
df

# 查看目录及子目录大小
du -H -h

# 查看当前目录下各个文件, 文件夹占了多少空间, 不会递归
du -sh *

```

wc命令
```shell
# 查看文件里有多少行
wc -l filename
 
# 看文件里有多少个word
wc -w filename

# 文件里最长的那一行是多少个字
wc -L filename

# 统计字节数
wc -c
```

压缩命令
```shell
tar czvf xxx.tar 压缩目录
 
zip -r xxx.zip 压缩目录
```

解压缩命令
```shell
tar zxvf xxx.tar
 
# 解压到指定文件夹
tar zxvf xxx.tar -C /xxx/yyy/
 
unzip xxx.zip
```

cp,scp,mkdir
```shell
#复制
cp xxx.log

# 复制并强制覆盖同名文件
cp -f xxx.log

# 复制文件夹
cp -r xxx(源文件夹) yyy(目标文件夹)

# 远程复制
scp -P ssh端口 username@10.10.10.101:/home/username/xxx /home/xxx

# 级联创建目录
mkdir -p /xxx/yyy/zzz

# 批量创建文件夹, 会在test,main下都创建java, resources文件夹
mkdir -p src/{test,main}/{java,resources}
```

比较两个文件
```shell
diff -u 1.txt 2.txt
```

查看, 去除特殊字符
```shell
# 查看特殊字符
cat -v xxx.sh

# 去除特殊字符
sed -i 's/^M//g’ env.sh  去除文件的特殊字符, 比如^M:  需要这样输入: ctrl+v+enter
```

grep
```shell
# 反向匹配, 查找不包含xxx的内容
grep -v xxx

# 排除所有空行
grep -v '^$'

# 返回结果 2,则说明第二行是空行
grep -n “^$” 111.txt    

# 查询以abc开头的行
grep -n “^abc” 111.txt 

# 同时列出该词语出现在文章的第几行
grep 'xxx' -n xxx.log

# 计算一下该字串出现的次数
grep 'xxx' -c xxx.log

# 比对的时候，不计较大小写的不同
grep 'xxx' -i xxx.log
```

awk
```shell
# 以':' 为分隔符,如果第五域有user则输出该行
awk -F ':' '{if ($5 ~ /user/) print $0}' /etc/passwd 

# 统计单个文件中某个字符（串）(中文无效)出现的次数
awk -v RS='character' 'END {print --NR}' xxx.txt
```

find检索命令
```shell
# 在目录下找后缀是.mysql的文件
find /home/eagleye -name '*.mysql' -print

# 会从 /usr 目录开始往下找，找最近3天之内存取过的文件。
find /usr -atime 3 –print
  
# 会从 /usr 目录开始往下找，找最近5天之内修改过的文件。
find /usr -ctime 5 –print

# 会从 /doc 目录开始往下找，找jacky 的、文件名开头是 j的文件。  
find /doc -user jacky -name 'j*' –print

# 会从 /doc 目录开始往下找，找寻文件名是 ja 开头或者 ma开头的文件。
find /doc \( -name 'ja*' -o- -name 'ma*' \) –print

#  会从 /doc 目录开始往下找，找到凡是文件名结尾为 bak的文件，把它删除掉。-exec 选项是执行的意思，rm 是删除命令，{ } 表示文件名，“\;”是规定的命令结尾。 
find /doc -name '*bak' -exec rm {} \;

```

find详解
```shell
按照文件名搜索
-name：按照文件名搜索
-iname：按照文件名搜索，不区分文件名大小写
-inum：按照inode号搜索

find . -name abcde


按照文件大小搜索
-size[+|-]大小：按照指定大小搜索文件

# 在当前目录下搜索文件大小为28k的文件
find . -size 28k

# 在当前目录下搜索文件大小大于1k的文件
find . -size +1k


按照修改时间搜索
-atime[+|-]时间：按照文件访问时间搜索
-mtime[+|-]时间：按照文件数据修改时间搜索
-ctime[+|-]时间：按照文件状态修改时间搜索
说明：
-5：代表5天内修改的文件。
5：代表前5~6天，那一天修改的文件。
+5：代表6天前修改的文件。

逻辑运算符
1）-a：and逻辑与
2）-o：or逻辑或
3）-not：not 逻辑非

# 在当前目录下搜索大于2KB，并且文件类型是普通文件的文件
find . -size +2k -a -type f

# 在当前目录下搜索文件要么是cangls的文件，要么是bols的文件，两个都可以搜索到
find . -name cangls -o -name bols

# 在当前目录下搜索文件名不是cangls的文件
find . -not -name cangls
```

查看什么进程使用了该端口
```shell
lsof -i:port
```

iostat,磁盘IO情况监控
```shell
iostat -xz 1
 
# r/s, w/s, rkB/s, wkB/s：分别表示每秒读写次数和每秒读写数据量（千字节）。读写量过大，可能会引起性能问题。
# await：IO操作的平均等待时间，单位是毫秒。这是应用程序在和磁盘交互时，需要消耗的时间，包括IO等待和实际操作的耗时。如果这个数值过大，可能是硬件设备遇到了瓶颈或者出现故障。
# avgqu-sz：向设备发出的请求平均数量。如果这个数值大于1，可能是硬件设备已经饱和（部分前端硬件设备支持并行写入）。
# %util：设备利用率。这个数值表示设备的繁忙程度，经验值是如果超过60，可能会影响IO性能（可以参照IO操作平均等待时间）。如果到达100%，说明硬件设备已经饱和。
# 如果显示的是逻辑设备的数据，那么设备利用率不代表后端实际的硬件设备已经饱和。值得注意的是，即使IO性能不理想，也不一定意味这应用程序性能会不好，可以利用诸如预读取、写缓存等策略提升应用性能。
```

free,内存使用情况
```shell
free -m
eg:
 
     total       used       free     shared    buffers     cached
Mem:          1002        769        232          0         62        421
-/+ buffers/cache:          286        715
Swap:          1153          0       1153
 
第一部分Mem行:
total 内存总数: 1002M
used 已经使用的内存数: 769M
free 空闲的内存数: 232M
shared 当前已经废弃不用,总是0
buffers Buffer 缓存内存数: 62M
cached Page 缓存内存数:421M
 
关系：total(1002M) = used(769M) + free(232M)
 
第二部分(-/+ buffers/cache):
(-buffers/cache) used内存数：286M (指的第一部分Mem行中的used – buffers – cached)
(+buffers/cache) free内存数: 715M (指的第一部分Mem行中的free + buffers + cached)
 
可见-buffers/cache反映的是被程序实实在在吃掉的内存,而+buffers/cache反映的是可以挪用的内存总数.
 
第三部分是指交换分区

```

vmstat, 给定时间监控CPU使用率, 内存使用, 虚拟内存交互, IO读写
```shell
# 2表示每2秒采集一次状态信息, 1表示只采集一次(忽略既是一直采集)
vmstat 2 1
 
eg:
r b swpd free buff cache si so bi bo in cs us sy id wa
1 0 0 3499840 315836 3819660 0 0 0 1 2 0 0 0 100 0
0 0 0 3499584 315836 3819660 0 0 0 0 88 158 0 0 100 0
0 0 0 3499708 315836 3819660 0 0 0 2 86 162 0 0 100 0
0 0 0 3499708 315836 3819660 0 0 0 10 81 151 0 0 100 0
1 0 0 3499732 315836 3819660 0 0 0 2 83 154 0 0 100 0

r 表示运行队列(就是说多少个进程真的分配到CPU)，我测试的服务器目前CPU比较空闲，没什么程序在跑，当这个值超过了CPU数目，就会出现CPU瓶颈了。这个也和top的负载有关系，一般负载超过了3就比较高，超过了5就高，超过了10就不正常了，服务器的状态很危险。top的负载类似每秒的运行队列。如果运行队列过大，表示你的CPU很繁忙，一般会造成CPU使用率很高。
b 表示阻塞的进程,这个不多说，进程阻塞，大家懂的。
swpd 虚拟内存已使用的大小，如果大于0，表示你的机器物理内存不足了，如果不是程序内存泄露的原因，那么你该升级内存了或者把耗内存的任务迁移到其他机器。
free 空闲的物理内存的大小，我的机器内存总共8G，剩余3415M。
buff Linux/Unix系统是用来存储，目录里面有什么内容，权限等的缓存，我本机大概占用300多M
cache cache直接用来记忆我们打开的文件,给文件做缓冲，我本机大概占用300多M(这里是Linux/Unix的聪明之处，把空闲的物理内存的一部分拿来做文件和目录的缓存，是为了提高 程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用。)
si 每秒从磁盘读入虚拟内存的大小，如果这个值大于0，表示物理内存不够用或者内存泄露了，要查找耗内存进程解决掉。我的机器内存充裕，一切正常。
so 每秒虚拟内存写入磁盘的大小，如果这个值大于0，同上。
bi 块设备每秒接收的块数量，这里的块设备是指系统上所有的磁盘和其他块设备，默认块大小是1024byte，我本机上没什么IO操作，所以一直是0，但是我曾在处理拷贝大量数据(2-3T)的机器上看过可以达到140000/s，磁盘写入速度差不多140M每秒
bo 块设备每秒发送的块数量，例如我们读取文件，bo就要大于0。bi和bo一般都要接近0，不然就是IO过于频繁，需要调整。
in 每秒CPU的中断次数，包括时间中断
cs 每秒上下文切换次数，例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好，太大了，要考虑调低线程或者进程的数目,例如在apache和nginx这种web服务器中，我们一般做性能测试时会进行几千并发甚至几万并发的测试，选择web服务器的进程可以由进程或者线程的峰值一直下调，压测，直到cs到一个比较小的值，这个进程和线程数就是比较合适的值了。系统调用也是，每次调用系统函数，我们的代码就会进入内核空间，导致上下文切换，这个是很耗资源，也要尽量避免频繁调用系统函数。上下文切换次数过多表示你的CPU大部分浪费在上下文切换，导致CPU干正经事的时间少了，CPU没有充分利用，是不可取的。
us 用户CPU时间，我曾经在一个做加密解密很频繁的服务器上，可以看到us接近100,r运行队列达到80(机器在做压力测试，性能表现不佳)。
sy 系统CPU时间，如果太高，表示系统调用时间长，例如是IO操作频繁。
id 空闲 CPU时间，一般来说，id + us + sy = 100,一般我认为id是空闲CPU使用率，us是用户CPU使用率，sy是系统CPU使用率。
wt 等待IO CPU时间。
```
