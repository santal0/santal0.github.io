# Linux 命令速查

## 网络相关

- `ping`：测试网络连通性
- `ifconfig`：查看网络接口信息
- `netstat`：查看网络连接信息
- `traceroute`：跟踪路由路径
- `nslookup`：域名解析
- `dig`：DNS 记录查询
- `whois`：查询域名信息
- `curl`：网络传输工具
- `wget`：下载网络资源
- `nmap`：网络扫描工具
- `tcpdump`：抓包工具
- `ss`：Socket 统计工具
- `arp`：地址解析协议工具
- `traceroute`：跟踪路由路径工具
- `mtr`：更好看的 traceroute
- `wireshark`：网络流量分析工具
- `tcpdump`：抓包工具
- `iftop`：监控网络流量工具

### 配置代理
使用 `proxychains` 连接 ftp 

#### 安装

```
sudo apt-get install -y proxychains4
```
或者
```
git clone git@github.com:rofl0r/proxychains-ng.git
sudo make
sudo make install
sudo make install-config
```
安装完之后可以找到 `/etc/proxychains.conf` 或 `/etc/proxychains4.conf` 文件进行修改，一般请求下翻到最后一段修改代理服务器配置即可。

## 最常用命令

1. 显示当前路径：pwd

2. 转换路径：cd  参数如下

   

   | 绝对路径 | 相对路径 | 祖先 | 默认值 | 当前 |      |
   | -------- | -------- | ---- | ------ | ---- | ---- |
   |          |          | ..   | ~      | .    |      |

3. 列举文件以及目录：ls

4. mkdir：新建目录

5. cat：显示文本内容

6. mv：移动

7. rm：删除文件

8. rmdir：删除目录

9. touch：后面加文件名，新建文件

10. find：-path ' '  -name   -type

    如： find -path '**/test/.py' 在test目录中寻找py文件

    find . -name a -type d 在当前文件夹内寻找名称为a的文件夹

11. locate：查询

12. grep：加“”，‘’，或无引号，查找包含关键字的行

    “^关键字”以关键字开头的行  "关键字$"以关键字结尾的行
    
13. chmod更改权限 用ls -l 列出第一列 三位为一组，分别指示：所有者的权限 、所属组的权限、其他人的权限 r读取，w写入，x执行

    例：chmod 777 html  （数字参数二进制形式分别控制九个位置，1表示打开权限，可以用r4 w2 x1来求和） 这个命令表示将html的所有者权限改为允许读、写、运行（4+2+1<<6），所属组权限改为允许读、写、运行（4+2+1<<3），其他人权限改为允许读、写、运行（4+2+1<<0）

    例：chmod u/g/o +/-/= rwx file     先是代号u所有者，g所有组，o其他人  +为增加权限 -为减少权限 =为更改权限 r读取，w写入，x执行 后加文件名

    如：chmod u=rw html/  给html目录权限设置为rw

14. 

### 命令检查与讲解

1. tldr：后面加命令，生成一个简洁的包含案例的用法列表

2. man:后面加命令，讲解命令

   

3. shellcheck：后面加.sh，检查错误

   如：shellcheck a.sh

## 运行

1. python：后面加文件名，解释一个python文件

2. gcc  -c（只编译不生成可执行文件）  -g（表示启用gdb）-E（只进行编译预处理（即生成.i的一步）） -S（生成汇编.s）  完整名称 -o（后方加生成文件名，或者-o2） 可执行文件名，需要扩展名 -Wall  如果有math等，应有-lm

   示例：gcc test.c -o test.exe -Wall

3. ./全名称，运行可执行文件

4. vim：加文件名，进入vim

## 语句

1. 赋值：变量名=  ;生成一个不确定类型的变量

2. 引用变量： $变量名

3. 循环

4. {逗号隔开的一列（每一个）或者..隔开的两个（从前一个到后一个）}

   如：touch {a..z}.txt 生成a.txt,b.txt......y.txt,z.txt

   touch {a,b}.txt 生成a.txt,b.txt

5. 通配符*所有，?一位

## 流

1. 改变 