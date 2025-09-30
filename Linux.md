# Linux 嵌入式开发

## 权限

### 权限的查看

```bash
ls -l <filename>
```

- r：可读 (read)
- w：可写 (write)
- x：可执行 (execute)

### 更改权限chmod

- 通过符号方式更改权限：

  ```bash
  chmod u+x 文件名   # 给文件拥有者添加执行权限
  chmod g-w 文件名   # 移除所属组的写权限
  chmod o+r 文件名   # 给其他用户添加读权限
  ```
- 通过数字方式更改权限：

  ```bash
  chmod 755 文件名   # 拥有者(rwx)，组(r-x)，其他用户(r-x)
  chmod 644 文件名   # 拥有者(rw-)，组(r--)，其他用户(r--)
  ```
- r = 4
- w = 2
- x = 1

## 防火墙设置



常用命令

```bash
ufw status         # 查看防火墙状态
ufw enable         # 启用防火墙
ufw disable        # 关闭防火墙
ufw allow 端口号   # 允许某端口通过
ufw deny 端口号    # 禁止某端口通过
ufw delete allow 端口号   # 删除允许规则
ufw delete deny 端口号    # 删除禁止规则
ufw reset          # 重置所有防火墙规则
ufw logging on     # 开启日志
ufw logging off    # 关闭日志
```
## 开机脚本 && 脚本执行顺序
编辑 `/etc/rc.local` 文件即可
```bash
/home/root/time.sh &   # 后台执行
/home/root/my_start.sh  # 阻塞执行      
exit 0        
```

## 显示进程动态

```bash
ps
```
## 开关内核打印

```bash
echo 0 > /proc/sys/kernel/printk # 关闭内核打印
echo 8 > /proc/sys/kernel/printk # 恢复默认值
```
## 创建属于自己的命令

### 通过/bin/进行修改

将c语言文件传入gcc进行编译，假设获得的可执行文件为 `my_cmd`

将此文件拷贝到 `/bin/`
即可随时输入 `my_cmd`进行调用

```bash
gcc test.c -o my_cmd
cp my_cmd /bin/
my_cmd   #在任何地方可以执行
```

### 通过修改全局变量进行修改

```bash
gcc test.c -o my_cmd
export PATH=/test/my_cmd:$PATH   #临时修改
```

如果需要常驻此全局变量则需要将上面的命令追加到 `/.bashrc` 随后重新加载

```bash
vim ~/.bashrc #在文件当中追加相应的内容(export PATH=/test/my_cmd:$PATH)
source ~/.bashrc
```

## make && makefile

1. make是一种编译辅助工具。先前通过gcc编译，但是gcc很麻烦，于是使用make进行编译 。
2. make会按照设置内容检测哪些文件需要再次编译、是否编译
3. makefile描述了整个工程的编译链接的一些规则，必须命名为makefile

### makefile怎么写

如下就是一个最简单的makefile

```makefile
all:command,o    # all的依赖内容三command.o
    gcc command.o -o command
  
command,o:command.c   # 如果找不到command.o 则 使用command.c编译出command.o
	gcc -c command.c -o command.o
```

### 快速删除make生成的文件

在makefile增加clean

```makefile
clean:
	rm rf *.o command   #删除掉所有.o文件 删除掉编译出的可执行文件command
```

保存之后，bash输入

```bash
make clean 
```

即可执行clean当中所写的内容

### 伪目标

当clean的内容当中包含了不属于编译产物的文件（但是和makefile当中的同名）的时候，直接make会报错

在clean前进行伪目标的声明,通常来说clean默认都应当被手动声明为伪目标。

```makefile
.PHONY:clean
clean:
	rm -rf *.o hello
```

### 变量赋值 && 引用

makefile的变量是可以赋值的desuwa

#### 立马赋值

```makefile
var1:=aaa
var2:=$(var1)bbb
var1:=ccc
all:
	echo $(var2)   # 输出为aaabbb
```

#### 延迟赋值

延迟赋值以最后被指定的值为准

```makefile
var1=aaa
var2=$(var1)bbb
var1=ccc
all:
	echo $(var2)   #输出cccbbb
```

#### 选择性赋值

如果已经被赋值，则无效。否则赋值为右值

```makefile
var1:=aaa
var1?=bbb # 此时不会赋值

var2?=ccc # 此时被赋值为ccc
```

#### 追加赋值

```makefile
var1:=aaa
var1+=bbb   # 此时var1为  aaabbb
```

## 自动化变量

常用的自动化变量有  `$@` 、`$<`、 `$^`

- `$@` 代表规则中的目标文件
- `$<` 代表第一个依赖文件
- `$^` 代表所有的依赖文件

```makefile
var:=command.o main.o
command:$(var)
  gcc $^ -o command
  #等同于 gcc $^ -o $@
%.o:%.c
  gcc -c $< -o $@
clean:
  rm -rf *.o command
```

如上内容的含义是：
- command依赖command.o和main.o
- 通过gcc将所有依赖文件编译成command
- 任何以.o结尾的文件都可以通过对应的.c文件编译而成

## wildcard函数
功能是展开符合条件的文件列表

```makefile
SRCS:=$(wildcard *.c)  #将当前目录下所有.c文件赋值给SRCS
SRCS:=$(wildcard src/*.c)  #将src目录下所有.c文件赋值给SRCS
OBJS:=$(SRCS:.c=.o)  #将SRCS当中的.c
#替换为.o并赋值给OBJS
```
## notdir函数
功能是去掉路径，只保留文件名
```makefile
SRCS:=src/command.c src/main.c
OBJS:=$(notdir $(SRCS))  #OBJS=command.c main.c
```

## dir函数
功能是去掉文件名，只保留路径
```makefile
SRCS:=src/command.c src/main.c
DIRS:=$(dir $(SRCS))  #DIRS=src/ src/
```

## patsubst函数
功能是替换字符串
```makefile
SRCS:=src/command.c src/main.c
OBJS:=$(patsubst src/%.c,build/%.o,$(SRCS))  
# 将src/command.c 替换为 build/command.o
```
替换不会改变当前目录下的文件

## foreach函数
功能是对列表当中的每一项进行操作
```makefile
SRCS:=src/command.c src/main.c
OBJS:=$(patsubst src/%.c,build/%.o,$(SRCS))  
DIRS:=$(dir $(OBJS))  #DIRS=build/ 
ALLDIRS:=$(foreach dir,$(DIRS),$(shell mkdir -p $(dir)))
# 对DIRS当中的每一项进行mkdir -p操作
```

## 连接开发板

打开命令行输入
```bash
ls /dev/ttyUSB*
```
查看对应的设备号后，使用com软件进行连接
```bash
minicom -D /dev/ttyUSB0 -b 115200
screen /dev/ttyUSB0 115200
picocom -b 115200 /dev/ttyUSB0
```
或者使用SSH
```bash
ssh -oHostKeyAlgorithms=+ssh-rsa root@192.168.3.48
```

接下来就能进入运行在arm架构下的linux系统了

# Linux下的C语言编程

## 网络连接
### 开启和关闭无线网卡
```bash
sudo ifconfig wlan0 up   #开启无线网卡
sudo ifconfig wlan0 down #关闭无线网卡
```
### 扫描无线网络
```bash
sudo iwlist wlan0 scan  #扫描无线网络
```
### 连接无线网络
```bash
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf  #连接无线网络
```

### 开启DHCP
```bash
udhcpc -i wlan0 
```

### 开启和关闭有线网卡
```bash
sudo ifconfig eth0 up   #开启有线网卡
sudo ifconfig eth0 down #关闭有线网卡
```
### 连接有线网络
```bash
dhclient eth0   #连接有线网络
```

### 查看网络连接状态
```bash
ifconfig   #查看网络连接状态
```
## cout 和 cerr 的区别
- cout 是标准输出流，通常用于输出正常的信息。
- cerr 是标准错误流，通常用于输出错误信息。cerr 通常不进行缓冲，意味着错误信息会立即显示出来，而不会被延迟。
在使用程序输出重定向时，cout 的输出可以被重定向到文件或其他设备，而 cerr 的输出通常仍然显示在终端上，以确保错误信息不会被忽略。



## main函数的参数
```c
int main(int argc, char *argv[])
```
- argc：表示命令行参数的个数，至少为1，因为***程序名称本身也是一个参数***。
- argv：是一个***字符串数组***，包含了所有的命令行参数。argv[0]是程序名称，argv[1]是第一个参数，依此类推。


## 开关GPIO（系统级应用）
```c
#include <stdio.h>

void main(int argc, char *argv[])
{
    int cnt;
    if (argc != 2)
    {
        printf("Usage: %s <0|1>\n", argv[0]);
        return;
    }
    else
    {
        cnt = (argv[1][0]) - '0';
    }
    // 蜂鸣器开关cnt次
    while (cnt--)
    {
        char command[100];
        snprintf(command, sizeof(command), "echo 1 > /sys/class/leds/beep/brightness");
        system(command); // 设置蜂鸣器频率
        sleep(0.5);
        snprintf(command, sizeof(command), "echo 0 > /sys/class/leds/beep/brightness");
        system(command); // 设置蜂鸣器频率
        sleep(1);
    }
}
```
编译成功后执行代码内容即可。
其中echo 1 > /sys/class/leds/beep/brightness
是通过系统命令的方式控制GPIO口的高低电平（在这里被抽象成了一个文件），Linux驱动开发者使得其在写入1时输出高电平，写入0时输出低电平。


## 文件IO
文件 IO 是直接调用内核提供的系统调用函数，头文件是 unistd.h，标准 IO 是间接调用系统调用函数，头文件是 stdio.h，文件 IO 是依赖于 Linux 操作系统的，标准 IO 是不依赖操作系统的，所以在任何的操作系统下，使用标准 IO，也就是 C 库函数操作文件的方法都是相同的。

文件 IO 使用文件操作符 、标准 IO 使用流操作符。

对于文件 IO 来说，一切都是围绕文件操作符来进行的。在 Linux 系统中，所有打开的文件都有一个对
应的文件描述符。文件描述符的本质是一个非负整数，当我们打开一个文件时，系统会给我们分配一个文
件描述符。当我们对一个文件做读写操作的时候，我们使用 open 函数返回的这个文件描述符会标识该文件，
并将其作为参数传递给 read 或者 write 函数。
文件描述符从3开始 ， 0、1、2分别是标准输入、标准输出和标准错误输出。
### 描述符
* fd == 0 : 标准输入
* fd == 1 : 标准输出
* fd == 2 : 标准错误输出
剩下1024 - 3 个描述符可以被用户程序使用（不同平台默认可分配的描述符数量不同，可以通过 ulimit -n 查看和设置）
### open函数
open函数的原型如下：
```c
#include <fcntl.h>
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
```
- pathname：要打开的文件路径。
- flags：打开文件的标志，可以是以下值的按位或：
- O_RDONLY：只读方式打开文件。
- O_WRONLY：只写方式打开文件。
- O_RDWR：读写方式打开文件。
- O_CREAT：如果文件不存在则创建文件。
- O_TRUNC：如果文件存在则将文件长度截断为0。
- O_APPEND：以追加方式写入文件。
- flags参数必须指定一个访问模式（O_RDONLY、O_WRONLY 或 O_RDWR）中的一个，可以与其他标志组合使用。
- mode：文件的权限，当使用 O_CREAT 标志创建文件时需要指定该参数。常用的权限值有：
- S_IRUSR：文件所有者具有读权限。
- S_IWUSR：文件所有者具有写权限。
- S_IRGRP：文件所属组具有读权限。
- S_IWGRP：文件所属组具有写权限。
- S_IROTH：其他用户具有读权限。
- S_IWOTH：其他用户具有写权限。
- 返回值：成功时返回文件描述符，失败时返回-1，并设置 errno 变量以指示错误原因。

### close函数
关闭open打开的文件，参数为open返回的文件描述符
```c
#include <unistd.h>
int close(int fd);
```
- fd：要关闭的文件描述符。
- 返回值：成功时返回0，失败时返回-1，并设置 errno 变量
- 以指示错误原因。
- 关闭文件描述符后，该文件描述符将不再有效，不能再用于读写操作。
- 关闭文件描述符是一个重要的资源管理操作，确保及时关闭不再使用的文件描述符可以避免资源泄漏。
- 在程序结束时，操作系统会自动关闭所有打开的文件描述符，但最好在不再需要文件时显式关闭它们。
- 如果一个文件描述符被多次关闭，后续的关闭操作将失败，并返回-1。
- 关闭一个文件描述符不会影响其他指向同一文件的文件描述符。
### read函数
读取文件内容，参数fd为open返回的文件描述符 buf为存储读取内容的缓冲区，count为要读取的字节数
#### ssize_t 类型
表示字节数或错误码
* 正数：表示实际读取/写入的字节数
* -1：表示操作失败（错误）
```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t count);

```
### write函数
写入文件内容，参数fd为open返回的文件描述符 buf为存储写入内容的缓冲区，count为要写入的字节数

```c
#include <unistd.h>
ssize_t write(int fd, const void *buf, size_t count);
```
### lseek函数
移动文件指针，参数fd为open返回的文件描述符 offset为偏移量
返回值为新的文件指针位置，失败返回-1 
```c
#include <unistd.h>
off_t lseek(int fd, off_t offset, int whence);
```
whence参数指定偏移的起始位置，可以是以下值之一：
- SEEK_SET：文件开头。
- SEEK_CUR：当前位置。
- SEEK_END：文件结尾。

## 目录IO
### mkdir函数
创建目录
```c
#include <sys/stat.h>
#include <sys/types.h>
int mkdir(const char *pathname, mode_t mode);
```
- pathname：要创建的目录路径。
- mode：目录的权限，通常使用八进制表示，例如 0755。
- 返回值：成功时返回0，失败时返回-1，并设置 errno 变量
  